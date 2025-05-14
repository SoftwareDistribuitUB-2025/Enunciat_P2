# Session 4

## Objectives

- Learn how to use JWT (JSON Web Tokens) for authentication.
  - Implement an authentication system both on the frontend and backend.
- Learn how to use Django permissions for authorization.
- Continue advancing the implementation of the frontend and backend connection using the Django REST framework and Axios.
- Add a registration form [outside of the lab].

## Login

#### Backend

In his part, we will implement a login system using JWT (JSON Web Tokens) for authentication. First, we will enable JWT authentication in the backend.

1. Uncomment the following lines in `settings.py` to enable JWT authentication:

```python
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}
```

You will notice that we cannot access the API now. This is because we have set the default permission class to `IsAuthenticated`, which means that only authenticated users can access the API.

2. Uncomment the following lines in `battleship/urls.py` to enable JWT authentication:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path("schema/", SpectacularAPIView.as_view(), name="schema"),
    path("docs/", SpectacularSwaggerView.as_view(url_name="schema"),name="swagger-ui"),
    path(r'ht/', include('health_check.urls')),
    # path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    # path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path("api/v1/", include('battleship.api.urls'))
]
```

#### Frontend

In the `frontend`, we have a service called `auth.js` and a store called `authStore.js`. We will make all the authentication calls from the `auth.js` service. The `authStore.js` store will be used to store the authentication state and the user information.

You can see that `auth.js` already has all the necessary functions (`login`, `logout`, `refresh`, etc.), but they are only returning a `mockAccessToken`. We need to make a call to the backend to get the actual access and refresh tokens. Here is the code for the `login` function:

```javascript
  login(user) {
    return axios.post("/api/token/", {
      username: user.username,
      password: user.password,
    });
  }
```

The `login` is called from the `authStore.js` store as follows:

```javascript
login(user) {
    this.loading = true;
    this.error = null;

    return AuthService.login(user)
    .then((response) => {
        console.log("response", response);
        // response = JSON.parse(response); // this is a mock, in real case it will be axios response and we can comment this line
        this.username = user.username;
        this.accessToken = response.data.access;
        this.refreshToken = response.data.refresh;
        this.isAuthenticated = true;

        localStorage.setItem("username", this.username);
        localStorage.setItem("access", this.accessToken);
        localStorage.setItem("refresh", this.refreshToken);
    })
    .catch((error) => {
        console.log("error", error);
        this.error =
        error.response?.data?.detail || "Login failed. Try again.";
        this.isAuthenticated = false;
    })
    .finally(() => {
        this.loading = false;
    });
},

```

Let's break it down:

- We are calling the `login` function from the `auth.js` service, passing the user object as an argument.
- The `login` function makes a POST request to the `/api/token/` endpoint with the username and password.
- If the request is successful, we store the access and refresh tokens in the local storage and set the `isAuthenticated` flag to `true`.
- If the request fails, we set the `error` message and the `isAuthenticated` flag to `false`.
- Finally, we set the `loading` flag to `false` to indicate that the request is complete. Note that the `loading` flag is a reactive property that is used to show a "Logging in..." message in the UI while the request is being processed.
- The `accessToken` and `refreshToken` are stored in the local storage so that we can use them later to make authenticated requests to the backend.
- The `username` is also stored in the local storage so that we can use it to display the username in the `Game.vue` component as a welcome message.

#### Todo:

- Implement the remaining functions in the `auth.js` service to handle logout and refresh tokens.

### Using the token for other requests

Now that we have the access token, we need to use it to make authenticated requests to the backend. We can do this by adding an `Authorization` header to the request with the token. For example, the function `getAllPlayers()` function now will require the token to be passed in the header:

```javascript
  getAllPlayers() {
    // return this.getAxiosInstance().get("/api/v1/players/");
    return this.getAxiosInstance().get("/api/v1/players/", {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
  }
```

After this modification, we can see that the home page is now showing the players.

#### Setting the token in the axios instance

At the moment, we are setting the token in each request. This is not ideal because we will have to do this for each request. Instead, we can set the token in the axios instance so that it is automatically added to each request. To do this, we make the following changes in the `getAxiosInstance()` function in the `auth.js` service:

```javascript
  getAxiosInstance() {
    const apiUrl = import.meta.env.VITE_API_URL;
    const instance = axios.create({
      baseURL: apiUrl,
      headers: {
        Authorization: `Bearer ${this.getAccessToken()}`,
      },
    });

    instance.interceptors.response.use(
      (response) => response,
      async (error) => {
        if (error.response.status === 401 && this.isLoggedIn()) {
          try {
            const response = await this.refresh(this.getRefreshToken());
            localStorage.setItem("access", response.data.access);
            error.config.headers["Authorization"] =
              "Bearer " + response.data.access;
            return axios.request(error.config);
          } catch (err) {
            this.logout();
          }
        }
        return Promise.reject(error);
      }
    );
  }
```

Let's break it down:

- We create an axios instance with the base URL and the `Authorization` header set to the access token.
- We use an interceptor to handle the 401 error. If we get a 401 error and the user is logged in, we try to refresh the token using the `refresh` function.
- If the refresh is successful, we update the access token in the local storage and set the `Authorization` header in the request config.
- Finally, we return the request with the updated config. If the refresh fails, we call the `logout` function to log the user out.
- Now, we can make requests to the backend without having to set the token in each request. The axios instance will automatically add the token to each request.
- We can also use the `getAxiosInstance()` function to make requests to the backend. For example, in the `getAllPlayers()` function we can go back to use the calls as follows:

```javascript
  getAllPlayers() {
    return this.getAxiosInstance().get("/api/v1/players/");
  }
```

Then, in `services/api.js`, we can import the `auth.js` service and get the axios instance, which we can use to make API calls for the game logic:

```javascript
import AuthService from "@/services/auth.js";
const axiosInstance = AuthService.getAxiosInstance();
```

## Outside of the lab

### Registration

- Implement a registration form in the home page next to the login form.
- The registration form should have the following fields:
  - Username
  - Password
  - Confirm password
  - Email
- The registration form should validate the fields and show an error message if the fields are not valid.
- Implement the registration function in the `auth.js` service to make a POST request to the `/api/v1/users/` endpoint with the user data.
