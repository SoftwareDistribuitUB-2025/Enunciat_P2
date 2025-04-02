# Backend Initialization with DJango

## Creating a Python Virtual Environment

We will manage dependencies using the tool [Poetry](https://python-poetry.org/). This tool allows us to define the required libraries and create a Python virtual environment with these libraries and all their dependencies. This way, we avoid conflicts with other Python projects.

To initialize the project, use the command:

```bash
poetry new backend
```

which generates the basic structure of the **backend** project. The **pyproject.toml** file contains all the information about the project, initially just basic information such as the name, version, and description of our project. In our case, we are not creating a new Python package, but an application, so we will add the following line to the configuration file:

```toml
[tool.poetry]
package-mode = false
```

We can also delete the **backend** folder that contains the empty package initially created by Poetry.
From here, you need to enter the new directory (where the `pyproject.toml` file is located) and add the necessary packages.  
We'll add the main packages using the command:

```bash
poetry add django djangorestframework drf-spectacular
```

Similarly, we can also add packages that are only needed for the development phase, such as those used to run tests on the code or check its correctness.  
We can add these dependencies with the command:

```bash
poetry add pytest pylint allure-pytest --group dev
```

To work within the virtual environment managed by Poetry, we can do it in two ways:

1. Activate the environment with `poetry shell`, which will make the Python virtual environment used for everything we do from that point on, until we run `deactivate`.
2. Explicitly use the environment by adding `poetry run` before each command you want to execute within the virtual environment. For example, `poetry run python --version` would run the `python --version` command inside the virtual environment.

Remember to use one of these methods throughout the rest of the guide.

## Initialize the Django project

[DJango](https://www.djangoproject.com/) is a framework for developing Python applications. To create the initial project, you need to run the following command (make sure the **backend** folder has been deleted in the previous step, or you will run into conflicts):

```bash
django-admin startproject backend
```

This command will have created a new **backend** folder, containing a **manage.py** file (among other things).  
The **manage.py** file allows us to interact with Django. We'll use it next to prepare the database and start the project.

### Configuration

All Django configuration is done through a configuration file, which you can find at  
`/backend/backend/backend/settings.py`. As we add components to our project, we’ll need to update this file accordingly.

We can add functionality to Django via **applications**, either ones we create or existing ones.  
To start, we’ll tell Django to use two applications we previously installed with Django.  
To do this, modify the **INSTALLED_APPS** list in the configuration file by adding `rest_framework` and `drf_spectacular` at the end:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'drf_spectacular',
]
```

### Database creation

Django provides a powerful database migration management system, which allows us to handle the creation and updating of the database to fit our application, as well as any existing applications we add.
We’ll explore migrations in practice later on—for now, we’ll simply apply the existing ones with the command:

```bash
python manage.py migrate
```

### Start the server

Once the database is created, we can check that our server is working. To start the server, run the command:

```bash
python manage.py runserver
```

If everything went well, you should be able to access the address
[http://127.0.0.1:8000/](http://127.0.0.1:8000/) with a browser, and see the DJango home page.

## Add a new application

Once we have our basic application working, now...
