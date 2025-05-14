# Sessió 4

## Objectius

* Aprendre a utilitzar JWT (JSON Web Tokens) per a l'autenticació.
  * Implementar un sistema d'autenticació tant en el frontend com en el backend.
* Aprendre a utilitzar els permisos de Django per a l'autorització.
* Continuar avançant en la implementació de la connexió entre el frontend i el backend utilitzant Django REST framework i Axios.
* Afegir un formulari de registre \[fora del laboratori].

## Inici de sessió

#### Backend

En aquesta part, implementarem un sistema d'inici de sessió utilitzant JWT (JSON Web Tokens) per a l'autenticació. Primer, habilitarem l'autenticació amb JWT en el backend.

1. Descomenta les següents línies a `settings.py` per habilitar l'autenticació JWT:

```python
REST_FRAMEWORK = {
    # Utilitza els permisos estàndard de Django `django.contrib.auth`,
    # o permet accés només de lectura per a usuaris no autenticats.
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

T'adonaràs que ara no podem accedir a l'API. Això és perquè hem establert la classe de permís per defecte com a `IsAuthenticated`, la qual cosa vol dir que només els usuaris autenticats poden accedir a l'API. Revisa la informació sobre permisos d'accés de l'API en [aquest enllaç](https://www.django-rest-framework.org/api-guide/permissions/). Fixa't que podem limitar l'accés **globalment** a nivell de recurs (Mètode + URL) o d'**instància/objecte** (un element en concret). Per exemple, podem limitar globalment el llistat s'usuaris a tothom que no sigui administrador, o permetre que un usuari pugui modificar **només** el registre corresponent al seu pròpi usuari (instància de User). Finalment, fixeu-vos que a part dels permisos per defecte, es **poden crear nous permisos** que s'adaptin a necessitats específiques de l'aplicació.

A banda d'assignar el tipus de permís per defecte, cada vista pot assignar un o més permisos. Per fer-ho, cal assignar les classes de permisos a la pròpia vista:

```python
from rest_framework import viewsets, permissions
from django.contrib.auth.models import User
from .serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ['username', 'email']
    permission_classes = [permissions.IsAdminUser]
```

La línia `permission_classes = [permissions.IsAdminUser]` indica que per accedir a aquesta vista caldrà ser administrador, i que per tant, un usuari normal o una persona no autenticada no hi podrà accedir. Els permisos es poden combinar per obtenir-ne de nous, utilitzant els operadors lògics i `|`, o `&` i/o no `~`. Per exemple, si assignem `permission_classes = [permissions.IsAdminUser|permissions.ReadOnly]`, obtindrem un permís d'accés en el qual els administradors tindran accés total, i la resta d'usuaris (autenticats o no) tindràn només accés de lectura (no podran fer ni POST, PUT, PATCH ni DELETE).


2. Descomenta les següents línies a `battleship/urls.py` per habilitar l'autenticació JWT:

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

En el `frontend`, tenim un servei anomenat `auth.js` i un magatzem anomenat `authStore.js`. Farem totes les crides d'autenticació des del servei `auth.js`. El `authStore.js` s'utilitzarà per emmagatzemar l'estat d'autenticació i la informació de l'usuari.

Pots veure que `auth.js` ja té totes les funcions necessàries (`login`, `logout`, `refresh`, etc.), però ara només estan retornant un `mockAccessToken`. Hem de fer una crida al backend per obtenir els tokens d'accés i de refresc reals. Aquí tens el codi de la funció `login`:

```javascript
  login(user) {
    return axios.post("/api/token/", {
      username: user.username,
      password: user.password,
    });
  }
```

La funció `login` és cridada des del `authStore.js` de la següent manera:

```javascript
login(user) {
    this.loading = true;
    this.error = null;

    return AuthService.login(user)
    .then((response) => {
        console.log("response", response);
        // response = JSON.parse(response); // això era una simulació, en el cas real és una resposta d'axios i podem comentar aquesta línia
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
        error.response?.data?.detail || "Error d'inici de sessió. Torna-ho a intentar.";
        this.isAuthenticated = false;
    })
    .finally(() => {
        this.loading = false;
    });
},
```

Desglossem-ho:

* Estem cridant la funció `login` del servei `auth.js`, passant l'objecte `user` com a argument.
* La funció `login` fa una petició POST a l'endpoint `/api/token/` amb el nom d'usuari i la contrasenya.
* Si la petició té èxit, emmagatzemem els tokens d'accés i de refresc al `localStorage` i establim el `isAuthenticated` a `true`.
* Si la petició falla, establim el missatge d'error i el `isAuthenticated` a `false`.
* Finalment, establim el `loading` a `false` per indicar que la petició ha finalitzat. Aquest valor s'utilitza per mostrar un missatge de "Iniciant sessió..." a la interfície.
* Els `accessToken` i `refreshToken` es guarden al `localStorage` per utilitzar-los més tard en peticions autenticades al backend.
* El `username` també es guarda per poder-lo mostrar al component `Game.vue` com a missatge de benvinguda.

#### Pendents:

* Implementar les funcions restants al servei `auth.js` per gestionar el tancament de sessió i la renovació de tokens.

### Utilitzar el token per a altres peticions

Ara que tenim el token d'accés, hem d'utilitzar-lo per fer peticions autenticades al backend. Ho podem fer afegint una capçalera `Authorization` a la petició amb el token. Per exemple, la funció `getAllPlayers()` ara necessitarà el token a la capçalera:

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

Després d’aquesta modificació, veurem que la pàgina inicial mostra els jugadors.

#### Establir el token a la instància d’axios

Ara mateix estem establint el token a cada petició. Això no és ideal perquè ho hauríem de fer a cada vegada. En comptes d’això, podem establir el token a la instància d’axios perquè s’afegeixi automàticament a totes les peticions. Per fer-ho, modifiquem la funció `getAxiosInstance()` al servei `auth.js`:

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

Desglossem-ho:

* Creem una instància d’axios amb la URL base i la capçalera `Authorization` amb el token d’accés.
* Utilitzem un interceptor per gestionar errors 401. Si rebem un error 401 i l’usuari està autenticat, intentem renovar el token amb la funció `refresh`.
* Si la renovació té èxit, actualitzem el token al `localStorage` i establim la capçalera `Authorization` amb el nou token.
* Finalment, fem la petició de nou amb la configuració actualitzada. Si la renovació falla, fem servir `logout()` per desconnectar l’usuari.
* Ara podem fer peticions al backend sense haver d’afegir el token manualment cada vegada. La instància d’axios l’afegeix automàticament.
* També podem utilitzar `getAxiosInstance()` per fer peticions al backend. Per exemple, a la funció `getAllPlayers()` podem tornar a escriure-la així:

```javascript
  getAllPlayers() {
    return this.getAxiosInstance().get("/api/v1/players/");
  }
```

Llavors, a `services/api.js`, podem importar el servei `auth.js` i obtenir la instància d’axios per fer crides API de la lògica del joc:

```javascript
import AuthService from "@/services/auth.js";
const axiosInstance = AuthService.getAxiosInstance();
```

## Fora del laboratori

### Registre

* Implementa un formulari de registre a la pàgina principal al costat del formulari d’inici de sessió.
* El formulari de registre ha de tenir els següents camps:

  * Nom d’usuari
  * Contrasenya
  * Confirmació de la contrasenya
  * Correu electrònic
* El formulari ha de validar els camps i mostrar un missatge d’error si no són vàlids.
* Implementa la funció de registre al servei `auth.js` per fer una petició POST a l’endpoint `/api/v1/users/` amb les dades de l’usuari.

> **Nota:** A l’hora de crear usuaris a Django, és fonamental seguir bones pràctiques per garantir la seguretat i la integritat del sistema. En cap cas s’han de desar les contrasenyes en text pla (en clar); Django proporciona mecanismes integrats per gestionar contrasenyes de manera segura mitjançant hashing. Per crear usuaris correctament, s’ha d'utilitzar el mètode `create_user()` proporcionat pel `UserManager`, ja que aquest s'encarrega d’encriptar automàticament la contrasenya abans de desar-la a la base de dades. A més, per crear superusuaris amb privilegis d’administració, s’ha d’utilitzar el mètode `create_superuser()`. Django també ofereix una API completa per gestionar usuaris, autenticació i permisos a través del mòdul `django.contrib.auth`, com es pot consultar a la [documentació oficial](https://docs.djangoproject.com/en/5.2/ref/contrib/auth/). Aquesta API inclou funcions per autenticar usuaris (`authenticate()`), iniciar i tancar sessions (`login()`, `logout()`), comprovar permisos (`has_perm()`, `has_module_perms()`), així com per gestionar canvis de contrasenya i recuperació de comptes. Adoptar aquestes pràctiques ajuda a construir aplicacions segures i mantenibles.
