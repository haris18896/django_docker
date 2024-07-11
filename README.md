# `Django Docker`

### Authenticate with Docker HUB
1. Go to [hub.docker.com](https://hub.docker.com) and login
2. Go to the `Account Settings` from your profile
3. Navigate to the `security`
4. Create new access token and add description to it
5. Save the token so that you can re-use it later I'm saving it in my slack account
6. go to the GitHub project settings => secrets === and add secrets
    * DOCKERHUB_USER
    * DOCKERHUB_TOKEN

### Using Docker with Django
1. Create a Docker file
2. choose a base image
3. install dependencies
4. setup users
5. configure docker compose
6. Define Services
    * Name
    * Mapping
    * Volume Map
7. Run all commands through docker compose `docker-compose run --rm app sh -c "python manage.py collectstatic"`

### Requirements
1. Create a `requirements.txt` file in the root of your repository
   * we can automatically add dependencies to the requirements by following the below commands
   * `python -m venv venv`
   * `source venv/bin/activate`
   * `pip freeze > requirements.txt` 
   * And to install the requirements `pip install -r requirements.txt`

2. create a docker file in the root of your repository
3. you can find different versions of python image in the [Python-Image-hub.docker.com](https://hub.docker.com/_/python)

```Dockerfile
FROM python:3.12.4-alpine3.20
LABEL maintainer='github.com/haris18896'

ENV PYTHONBUFFERED 1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./requirements.dev.txt /tmp/requirements.dev.txt

COPY ./app /app

WORKDIR /app

EXPOSE 8000

ARG DEV=false

RUN python -m venv venv /py && \
    /py/bin/pip install --upgrade pip && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ $DEV = "true" ]; \
      then /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    adduser \
    --disabled-password \
    --no-create-home \
    django-user  # name of django user

ENV PATH="/py/bin:$PATH"

USER django-user
```

1. Next we are adding `.dockerignore` file to the root directory
2. After configuring run the following command
   * docker build .


### Docker Compose
1. create a `docker-compose.yml` file in the root directory
2. after configuring run the following command `docker-compose build`
3. add `requirements.dev.txt` to the root directory, it's for local development, these aren't gonna be needed on image
4. Run `docker-compose build --build-arg DEV=true`
5. Run `docker-compose run --rm app sh -c "flake8"`

```yml
version: "3.12.4"
name: "recipe-app-api"

services:
  app:
    build:
      context: .
      args:
         - DEV=true
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
```


### Configuring flake8
1. Create a `.flake8` file in the `app` directory
2. after adding the configurations to the `.flake8` file run the following command
   * `docker-compose run --rm app sh -c "flake8"`


```.flake8
[flake8]
exclude =
    migrations,
    __pycache__,
    manage.py,
    settings.py
```

### Creating a django project in app
1. Run the following command to create a django project
   * `docker-compose run --rm app sh -c "django-admin startproject app ."`
2. To run our services run the following command
   * `docker-compose up`
3. After that check the browser and run `127.0.0.1:8000` the app should run at this point

# `Configuring GitHub Actions`
* GitHub's actions are like automation tools Travis-CI, GitLab CI/CD, Jenkins
* Run jobs when code changes
* It handles Deployment, Code Linting, and Unit Tests

1. Create `.github/workflows/checks.yml` file in the root of your repository
2. After adding the checks.yml file make sure the username and password matches the one you have added as a secret in the repository settings in Github
3. After that add and commit you changes and push to `main` branch
4. after that check the `Acitons` in your github repository, your actions should have been configured their

```yml
---
name: Checks

on: [push] # Trigger

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login/action@v3.2.0
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build Docker image
        run: docker-compose build --build-arg DEV=true
      - name: Test
        run: docker-compose run --rm app sh -c "python manage.py test"
      - name: Lint
        run: docker-compose run --rm app sh -c "flake8"
```


# `Django Test`
1. Based on the `unittest` library
2. Django adds features to the `unittest` library
   * Test client - which is dummy web browser that you can use to make a request to your project
   * Simulate authentication
   * Temporary Database
3. Django REST Framework adds features
   * API test client - same as dummy web browser
4. We can either use `tests.py` file that already comes with django app when we initialize it or we can use `tests` directory in that case we will have to add prefix `test_`to each module
5. Test directory must contain `__init__.py`
6. Test code that uses the DB
7. Django uses specific database for test

### Django Test Classes
1. `SimpleTestCase`
   * No database integration
   * useful if no database is required for your test
   * save time executing tests
2. `TestCase`
   * database integrations is required
   * useful for testing code that uses the database

### Writing Tests
1. Import test class
   * SimpleTestCase or TestCase
2. Import objects to test
3. Define test class
4. Add test methods (make sure to add `test_` prefix to every method so that it can be picked by the Django testing)


### Mocking
1. override or change behaviour of dependencies
2. Avoid unintended side effects
3. Isolate code being tested

### Why use mocking?
1. Avoid relaying on external services
   * Can't guarantee they will be available
   * Make test unpredictable and inconsistent
2. Avoid unintended consequences
   * Accidentally sending emails
   * Overloading external services
3. Speeds up tests

### How to mock code
1. You can use se `uinttest.mock` library
   * `MagicMock/Mock` - it replaces your real objects with mock objects
   * `patch` - Overrides code for testing


# `Configuring DataBase`
1. Our docker-compose will have 2 services
   * Database Service
   * App Service
2. Django needs to know about
   * Engine (type of database)
   * Hostname (IP or domain name for database)
   * Port (default 5432)
   * Database Name
   * Username
   * password
3. We will have to define it in the `settings.py`

### Psycopg2
1. The package that is needed for the django to be connected to database
2. Most popular PostgreSQL adaptor for python
3. Supported by Django
4. Packages
   * psycopg2-binary
   * psycopg2

* Run `docker-compose up` to initiate the db and volume in the docker
* Adding the postgresql-client to the `dockerfile` 
```dockerfile
FROM python:3.12.4-alpine3.20
LABEL maintainer='github.com/haris18896'

ENV PYTHONBUFFERED 1

COPY ./requirements.txt /tmp/requirements.txt
COPY ./requirements.dev.txt /tmp/requirements.dev.txt

COPY ./app /app

WORKDIR /app

EXPOSE 8000

ARG DEV=false

RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    apk add --update --no-cache postgresql-client && \
    apk add --update --no-cache --virtual .tmp-build-deps \
        build-base postgresql-dev musl-dev && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    if [ "$DEV" = "true" ]; then \
      /py/bin/pip install -r /tmp/requirements.dev.txt ; \
    fi && \
    rm -rf /tmp && \
    apk del .tmp-build-deps && \
    adduser --disabled-password --no-create-home django-user

ENV PATH="/py/bin:$PATH"

USER django-user
```

* After adding `psycopg2>=2.9.9` in to the `requirements.txt` file
  * Run `docker-compose down` to stop the Docker image
  * And then build the image `docker-compose build` it will install the psycopg2 and will build our image
* Change the `DATABASES` in `settings.py`

```py
import os
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': os.environ.get('DB_HOST'),
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASS')
    }
}
```

### Problem with Docker compose
1. Using `depends_on` ensures service starts
   * But Doesn't ensure application is running
2. This issue appears 
   * when we are running `docker-compose` locally
   * Running on deployed environment


* with the above solution, when the docker runs the database service as it will run it first, once the database service stop starting it will start the django app service
* meanwhile the database service will start Postgres and the app service will start app service
* But at this point the database service has not completed starting the postgres service and it's not ready, on the other-hand the django app is ready and trying to connect to the database
* at this point we will get an error. and this error appears on the first time, soo we will have to restart the service and it will work

![dockerTimeLine.png](..%2F..%2F..%2F..%2F..%2FDesktop%2FdockerTimeLine.png)

#### `Solution`
1. Make Django `wait for db`
   * Check for database availability
   * Continue when database is ready
2. Create a Custom Django management command

* with the above solution, when the docker runs the database service as it will run it first, once the database service stop starting it will start the django app service
* meanwhile the database service will start Postgres and the app service will start app service
* Now at this point the Django-app service will wait for the database service to be ready, and it will check if it's ready
* and once the database is ready it will connect with the database

![newDockerTimeLine.png](..%2F..%2F..%2F..%2F..%2FDesktop%2FnewDockerTimeLine.png)

### Fixing Database Race Condition
1. Create a django app called `core`
2. From the `core` app 
   * we will delete `tests.py` file and will add `tests` directory
   * we will delete `views.py` because our core app will not serve any views
3. Add `core` app to the `settings.py` file

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',
]
```

### Adding Command to core to wait for DB
1. Create a file in the `core/management/commands/wait_for_db.py`
2. Create a file in the `core/tests/test_commands.py`

```py
# wait_for_db.py
"""
Django command to wait for the database to be available
"""
import time

from django.db.utils import OperationalError
from psycopg2 import OperationalError as Psycopg2OpError

from django.core.management.base import BaseCommand


class Command(BaseCommand):
    """Django command to wait for database"""

    def handle(self, *args, **options):
       self.stdout.write("Waiting for database....")
       db_up = False
       while db_up is False:
           try:
               self.check(databases=['default'])
               db_up = True
           except (Psycopg2OpError, OperationalError):
               self.stdout.write("Database unavailable, waiting 1 second...")
               time.sleep(1)

       self.stdout.write(self.style.SUCCESS('database available!'))
```

```py
# test_commands.py
"""
Test custom Django management commands.
"""
from unittest.mock import patch

from psycopg2 import OperationalError as Psycopg2OpError

from django.core.management import call_command
from django.db.utils import OperationalError
from django.test import SimpleTestCase


@patch('core.management.commands.wait_for_db.Command.check')
class CommandTests(SimpleTestCase):
    """Test commands."""

    def test_wait_for_db_ready(self, patched_check):
        """Test waiting for database if database ready."""
        patched_check.return_value = True

        call_command('wait_for_db')

        patched_check.assert_called_once_with(databases=['default'])

    @patch('time.sleep')
    def test_wait_for_db_delay(self, patched_sleep, patched_check):
        """Test waiting for database when getting OperationalError."""
        patched_check.side_effect = [Psycopg2OpError] * 2 + \
            [OperationalError] * 3 + [True]

        call_command('wait_for_db')

        self.assertEqual(patched_check.call_count, 6)
        patched_check.assert_called_with(databases=['default'])

```


### Django ORM
1. Object Relational Mapper (ORM)
2. Abstraction layer for data
   * Django handles database structure and changes
   * Focus on python code
   * Use any database (within reason)

# `Django User Model`
1. Foundation of the django auth system
2. Default user model
   * Username instead of email
   * Not easy to customise
3. Create a custom model for new projects
   * Allows for using email instead of username

### How to Customise user model
1. Create model
   * Base form `AbstractBaseUser` and `PermissionMixin`
2. Create custom manager
   * Used for CLI integration
3. Set `AUTH_USER_MODEL` in settings.py
4. Make migrations

* Below we are adding a test case for user model.
* At this point the test will fail because we haven't added any Custom User model, so the default user model will expect `username`

```py
# test_models.py
"""
Tests for models.
"""
from django.test import TestCase
from django.contrib.auth import get_user_model


class ModelTests(TestCase):
    """Test Models"""
    def test_create_user_with_email_successful(self):
        """Text creating a user with an email is successful"""
        email = 'test@example.com'
        password = 'testpass123'
        user = get_user_model().objects.create_user(email=email, password=password)


        self.assertEqual(user.email, email)
        self.assertTrue((user.check_password(password)))
```

* Then we are going to add a Custom User model

```py
# core/models.py
"""Database Models"""
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin
)


class User(AbstractBaseUser, PermissionsMixin):
    """User in the system."""
    email = models.EmailField(max_length=255, unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    USERNAME_FIELD = 'email'
```

* Once we create the `ModelTests` and CustomUserModel `User` we are going to create `UserManger` model in `core/models.py`
```py
 # core/models.py
"""Database Models"""
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin
)


class UserManager(BaseUserManager):
    """Manager for users"""
    def create_user(self, email, password=None, **extra_field):
        """Create, save and return a new user"""
        user  = self.model(email=email, **extra_field)
        user.set_password(password)
        user.save(using=self._db)

        return user


class User(AbstractBaseUser, PermissionsMixin):
    """User in the system."""
    email = models.EmailField(max_length=255, unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = UserManager()

    USERNAME_FIELD = 'email'
```


* Setting the `AUTH_USER_MODEL` configuration in `settings.py`
* And then make migrations `docker-compose run --rm app sh -c "python manage.py makemigrations"`

```py
# app/settings.py

# /........../
# /........../
# /........../
AUTH_USER_MODEL = 'core.User'
```

### Removing Volume

* Run `docker-compose run --rm app sh -c "python manage.py wait_for_db && python manage.py migrate"`
* if you see `django.db.migrations.exceptions.InconsistentMigrationHistory:`
  * then clear the data from our database
  * Run `docker volume ls`
  * Run `docker-compose down`
  * Remove the volume `docker volumne rm <volume-name>`
  * Then re-run the migrations in first step
  * and after that run the tests

### Add Feature to Normalize User Email
1. Adding Test for `email` entered and the expected email
2. To Normalize User Email

```py
   # models
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin
)

# update the UserManager
class UserManager(BaseUserManager):
    """Manager for users"""
    def create_user(self, email, password=None, **extra_fields):
        """Create, save and return a new user"""
        user  = self.model(email=self.normalize_email(email), **extra_fields)
        user.set_password(password)
        user.save(using=self._db)

        return user
```

```py
"""
Tests for models.
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
class ModelTests(TestCase):
    """Test Models"""
    
    # /****** test_create_user_with_email_successful(self):  *******/
    
    def test_new_user_email_normalized(self):
        """Test email is normalized for new users"""
        sample_emails = [
            ['test1@EXAMPLE.com', 'test1@example.com'],
            ['test2@Example.com', 'test2@example.com'],
            ['test3@example.COM', 'test3@example.com'],
        ]

        for email, expected in sample_emails:
            user = get_user_model().objects.create_user(email, 'sample123')
            self.assertEqual(user.email, expected)
```

### Add Require email address
```py
from django.test import TestCase
from django.contrib.auth import get_user_model
class ModelTests(TestCase):
    """Test Models"""
    
    # /****** test_create_user_with_email_successful(self):  *******/
    # /****** test_new_user_email_normalized(self):  *******/
    
    def test_new_user_without_email_raises_error(self):
        """Test that creating a user without an email raises a ValueError"""
        with self.assertRaises(ValueError):
            get_user_model().objects.create_user('', 'test123')
```

### Add Superuser Functionality
1. As Always first we will add the Test
2. And then we will write the function
3. and then run tests

```py
# Test
from django.test import TestCase
from django.contrib.auth import get_user_model
class ModelTests(TestCase):
    """Test Models"""
    
    # /****** test_create_user_with_email_successful(self):  *******/
    # /****** test_new_user_email_normalized(self):  *******/
    # /****** test_new_user_without_email_raises_error(self): ******/
    def test_create_superuser(self):
        """Test creating a superuser"""
        user = get_user_model().objects.create_superuser(
            'test@example.com',
            'test123'
        )

        self.assertTrue(user.is_superuser)
        self.assertTrue(user.is_staff)
```

```py
# UserModel
"""Database Models"""
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin
)

class UserManager(BaseUserManager):
    # /****** def create_user(self, email, password=None, **extra_fields): ******/
    
    def create_superuser(self, email, password):
        """Create, save and return a new superuser"""
        user = self.create_user(email, password)
        user.is_staff = True
        user.is_superuser = True
        user.save(using=self._db)

        return user
```

* After that run the below command to create a new superuser
  * `docker-compose run --rm app sh -c "python manage.py createsuperuser"`


# `Setup Django Admin`

### What is the Django Admin?
1. A Graphical User Interface for models
   * Create, Read, Update, Delete
2. Very little coding required 
3. How to Enable
   * Enabled per model
   * inside `admin.py`
      * `admin.site.register(Recipe)`
4. Customising
   * Create a class based off `ModelAdmin` or `UserAdmin`
   * Override/set class variables 
   * We can also change list of Objects
5. Changing List of Objects
   * `Ordering`: Changes order items appear
   * `list_display`: fields to appear in list
6. Add/update page
   * `fieldsets`: Control layout of page
   * `readonly_fields`: fields that cannot be changed
7. Add Page
   * `add_fieldssets`

* First we are going to create a test
* And then we will write Django admin functions

```py
# core/tests/test_admin.py
"""
Tests for the Django admin modifications
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse
from django.test import Client


class AdminSiteTests(TestCase):
    """Tests for Django Admin."""

    def setUp(self):
        """Create user and client"""
        self.client = Client()
        self.admin_user = get_user_model().objects.create_superuser(
            email='admin@example.com',
            password='incorrect'
        )
        self.client.force_login(self.admin_user)
        self.user = get_user_model().objects.create_user(
            email='user@example.com',
            password='incorrect',
            name='Test User'
        )

    def test_users_list(self):
        """Test that users are listed on page."""
        url = reverse('admin:core_user_changelist')
        res = self.client.get(url)

        self.assertContains(res, self.user.name)
        self.assertContains(res, self.user.email)
```

```py
# core/admin.py
"""Django admin customization."""

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin

from . import models

class UserAdmin(BaseUserAdmin):
    """Define the admin pages for users"""
    ordering = ['id']
    list_display = ['email', 'name']


admin.site.register(models.User, UserAdmin)
```

* At this point we can see Users in our django admin
* But we are unable to move to the next screen when clicking on the email of the user

* For that we are going write test first
* and then we will update the `admin.py`

```py
# core/tests/test_admin.py
"""
Tests for the Django admin modifications
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse
from django.test import Client

class AdminSiteTests(TestCase):
    # / ***** def setUp(self): ***** /
    # / *****  def test_users_list(self): ***** /
    
    def test_edit_user_page(self):
        """Test the edit user page works"""
        url = reverse('admin:core_user_change', args=[self.user.id])
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)
```

```py
"""Django admin customization."""

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _

from . import models


class UserAdmin(BaseUserAdmin):
    """Define the admin pages for users"""

    ordering = ["id"]
    list_display = ["email", "name"]
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (
            _('Permissions'),
            {
                'fields': (
                    'is_active',
                    'is_staff',
                    'is_superuser',
                )
            }
        ),
        (_('Important dates'), {'fields': ('last_login',)})
    )
    readonly_fields = ['last_login']


admin.site.register(models.User, UserAdmin)
```

* Now customizing the admin page for changing user
* First we are going to write test for it
* And then we will update the Admin Model

```py
```py
# core/tests/test_admin.py
"""
Tests for the Django admin modifications
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse
from django.test import Client

class AdminSiteTests(TestCase):
    # / ***** def setUp(self): ***** /
    # / ***** def test_users_list(self): ***** /
    # / ***** def test_edit_user_page(self): ***** /

    def test_create_user_page(self):
        """Test the create user page works"""
        url = reverse('admin:core_user_add')
        res = self.client.get(url)

        self.assertEqual(res.status_code, 200)
```

```py
# core/test/test_admin.py
"""Django admin customization."""

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.utils.translation import gettext_lazy as _

from . import models


class UserAdmin(BaseUserAdmin):
    """Define the admin pages for users"""

    ordering = ["id"]
    list_display = ["email", "name"]
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        (
            _('Permissions'),
            {
                'fields': (
                    'is_active',
                    'is_staff',
                    'is_superuser',
                )
            }
        ),
        (_('Important dates'), {'fields': ('last_login',)})
    )
    readonly_fields = ['last_login']
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': (
                'email',
                'password1',
                'password2',
                'name',
                'is_active',
                'is_staff',
                'is_superuser'
            )
        }),
    )


admin.site.register(models.User, UserAdmin)
```


# `API Documentation`
1. Auto generate docs in Django REST Framework
   * `drf-spectacular`
2. It generates schema documentation
3. Browsable web interface
   * Make test requests
   * Handle authentication

* first add `drf-spectacular>=0.27.2` to the requirements
* Run the `docker-compose build`
* Add the `drf-spectacular` to the settings.py to installed apps

```py
# settings.py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "core",
    "drf_spectacular"
]

# / *****************
# / *****************
# / *****************

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema'
}
```

* After that we have to add the links to the URLs

```py
# app/app/urls.py
# from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path("admin/", admin.site.urls),
    # path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # # Optional UI:
    # path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    # path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]

```


# `Build User API`
1. In this section we are going to create a user API
   * User Registration
   * Create auth token
   * Viewing/Updating profile
2. Endpoints
   * `user/create/` this will be a `post` - request, Registering a new user
   * `user/token` this will be a `post` - request, Create a new token
   * `user/me/` this will be a `put/patch` request, Update profile
   * `user/me/` this will be a `get` request, View profile
3. We are going to create a new django-app for our user API 
   * Run `docker-compose run --rm app sh -c "python manage.py startapp user"`
4. Do some cleaning on the User project.
   * Remove `migrations` because we are going to have all the migrations in the `core` app
   * Remove `admin.py`
   * Remove `models.py`
   * Remove `test.py` and Create a `tests` directory


* Add the `User` app to the `settings.py`

### `Writing Tests for User API`
1. Create a `test_user_api.py` in the `user/tests` directory
2. Create a `test_user_api.py` in the `user/tests` directory

```py
"""
Tests for the user API
"""

from django.test import TestCase
from django.urls import reverse
from django.contrib.auth import get_user_model

from rest_framework.test import APIClient
from rest_framework import status

CREATE_USER_URL = reverse('user:create')

def create_user(**params):
    """Create and return a new user."""
    return get_user_model().objects.create_user(**params)

class PublicUserApiTests(TestCase):
    """Test the users API (public)"""

    def setUp(self):
        self.client = APIClient()

    def test_create_valid_user_success(self):
        """Test creating user with valid payload is successful"""
        payload = {
            'email': 'test@example.com',
            'password': 'testPass123',
            'name': 'Test Name'
        }

        res  = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_201_CREATED)
        user = get_user_model().objects.get(email=payload['email'])
        self.assertTrue(user.check_password(payload['password']))
        self.assertNotIn('password', res.data)

    def test_user_with_email_exists_error(self):
        """Test creating a user that already exists"""
        payload = {
            'email': 'test@example.com',
            'password': 'testPass123',
            'name': 'Test Name'
        }

        create_user(**payload)
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

    def test_password_too_short_error(self):
        """Test that the password must be more than 5 characters"""
        payload = {
            'email': 'test@example.com',
            'password': 'pw',
            'name': 'Test Name'
        }

        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

        user_exists = get_user_model().objects.filter(
            email=payload['email']
        ).exists()

        self.assertFalse(user_exists)
```

* After writing the tests we are going to write the API because the test going to fail without that
* For that we are going to create a new `Serializers` in the `user` app

```py
# user/serializers.py

"""
Serializers for the user API View
"""

from django.contrib.auth import get_user_model

from rest_framework import serializers


class UserSerializer(serializers.ModelSerializer):
    """Serializer for the users object"""

    class Meta:
        model = get_user_model()
        fields = ('email', 'password', 'name')
        extra_kwargs = {'password': {'write_only': True, 'min_length': 5}}

    def create(self, validated_data):
        """Create a new user with encrypted password and return it"""
        return get_user_model().objects.create_user(**validated_data)

```

* After writing the `seriliazers` we are going to write the `views` for the `user` app

```py
"""
Views for the user API
"""

from rest_framework import generics
from .serializers import UserSerializer


class CreateUserView(generics.CreateAPIView):
    """Create a new user in the system"""
    serializer_class = UserSerializer

```

* After writing the `views` we are going to add the `urls` for the `user` app

```py
"""
URL mappings for the user API.
"""

from django.urls import path
from . import views

app_name = 'user'

urlpatterns = [
    path('create/', views.CreateUserView.as_view(), name='create'),
]
```

* Once we have the `urls` we are going to connect the `user/urls` to the `app/urls`

```pyc
from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # # Optional UI:
    path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),

    path('api/user/', include('user.urls')),
]
```

### `Authentication`
1. Basic
    * Send Username and password with each request
    * `BasicAuthentication`
2. Token based authentication
   * Use a token in HTTP header
   * `TokenAuthentication`
   * `ObtainAuthToken`
3. JSON Web Token (JWT)
   * Use an access and refresh token
   * `SimpleJWT`
   * `JWTAuthentication`
4. Session Authentication
   * Use cookies
   * `SessionAuthentication`

* We are going to use `TokenAuthentication` for our API
* also make sure to add `rest_framework.authtoken` to the `INSTALLED_APPS` in the `settings.py`

1. First we are going to write tests for our `APIs`
2. After that we are going to write the `seriliazers` for the `create user, Token, and me endpoint` in the `user` app 
3. Then that we are going to write the `views` for the `create user, Token, and me endpoint` in the `user` app 
4. Then we are going to add the `urls` for the `create user, Token, and me endpoint` in the `user` app

```py
# Tests
"""
Tests for the user API.
"""
from django.test import TestCase
from django.contrib.auth import get_user_model
from django.urls import reverse

from rest_framework.test import APIClient
from rest_framework import status


CREATE_USER_URL = reverse('user:create')
TOKEN_URL = reverse('user:token')
ME_URL = reverse('user:me')


def create_user(**params):
    """Create and return a new user."""
    return get_user_model().objects.create_user(**params)


class PublicUserApiTests(TestCase):
    """Test the public features of the user API."""

    def setUp(self):
        self.client = APIClient()

    def test_create_user_success(self):
        """Test creating a user is successful."""
        payload = {
            'email': 'test@example.com',
            'password': 'testpass123',
            'name': 'Test Name',
        }
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_201_CREATED)
        user = get_user_model().objects.get(email=payload['email'])
        self.assertTrue(user.check_password(payload['password']))
        self.assertNotIn('password', res.data)

    def test_user_with_email_exists_error(self):
        """Test error returned if user with email exists."""
        payload = {
            'email': 'test@example.com',
            'password': 'testpass123',
            'name': 'Test Name',
        }
        create_user(**payload)
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

    def test_password_too_short_error(self):
        """Test an error is returned if password less than 5 chars."""
        payload = {
            'email': 'test@example.com',
            'password': 'pw',
            'name': 'Test name',
        }
        res = self.client.post(CREATE_USER_URL, payload)

        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)
        user_exists = get_user_model().objects.filter(
            email=payload['email']
        ).exists()
        self.assertFalse(user_exists)

    def test_create_token_for_user(self):
        """Test generates token for valid credentials."""
        user_details = {
            'name': 'Test Name',
            'email': 'test@example.com',
            'password': 'test-user-password123',
        }
        create_user(**user_details)

        payload = {
            'email': user_details['email'],
            'password': user_details['password'],
        }
        res = self.client.post(TOKEN_URL, payload)

        self.assertIn('token', res.data)
        self.assertEqual(res.status_code, status.HTTP_200_OK)

    def test_create_token_bad_credentials(self):
        """Test returns error if credentials invalid."""
        create_user(email='test@example.com', password='goodpass')

        payload = {'email': 'test@example.com', 'password': 'badpass'}
        res = self.client.post(TOKEN_URL, payload)

        self.assertNotIn('token', res.data)
        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

    def test_create_token_blank_password(self):
        """Test posting a blank password returns an error."""
        payload = {'email': 'test@example.com', 'password': ''}
        res = self.client.post(TOKEN_URL, payload)

        self.assertNotIn('token', res.data)
        self.assertEqual(res.status_code, status.HTTP_400_BAD_REQUEST)

    def test_retrieve_user_unauthorized(self):
        """Test authentication is required for users."""
        res = self.client.get(ME_URL)

        self.assertEqual(res.status_code, status.HTTP_401_UNAUTHORIZED)


class PrivateUserApiTests(TestCase):
    """Test API requests that require authentication."""

    def setUp(self):
        self.user = create_user(
            email='test@example.com',
            password='testpass123',
            name='Test Name',
        )
        self.client = APIClient()
        self.client.force_authenticate(user=self.user)

    def test_retrieve_profile_success(self):
        """Test retrieving profile for logged in user."""
        res = self.client.get(ME_URL)

        self.assertEqual(res.status_code, status.HTTP_200_OK)
        self.assertEqual(res.data, {
            'name': self.user.name,
            'email': self.user.email,
        })

    def test_post_me_not_allowed(self):
        """Test POST is not allowed for the me endpoint."""
        res = self.client.post(ME_URL, {})

        self.assertEqual(res.status_code, status.HTTP_405_METHOD_NOT_ALLOWED)

    def test_update_user_profile(self):
        """Test updating the user profile for the authenticated user."""
        payload = {'name': 'Updated name', 'password': 'newpassword123'}

        res = self.client.patch(ME_URL, payload)

        self.user.refresh_from_db()
        self.assertEqual(self.user.name, payload['name'])
        self.assertTrue(self.user.check_password(payload['password']))
        self.assertEqual(res.status_code, status.HTTP_200_OK)
```


```py
# Serializers
"""
Serializers for the user API View.
"""

from django.contrib.auth import (
    get_user_model,
    authenticate,
)
from django.utils.translation import gettext as _

from rest_framework import serializers


class UserSerializer(serializers.ModelSerializer):
    """Serializer for the user object."""

    class Meta:
        model = get_user_model()
        fields = ["email", "password", "name"]
        extra_kwargs = {"password": {"write_only": True, "min_length": 5}}

    def create(self, validated_data):
        """Create and return a user with encrypted password."""
        return get_user_model().objects.create_user(**validated_data)

    def update(self, instance, validated_data):
        """Update and return user."""
        password = validated_data.pop("password", None)
        user = super().update(instance, validated_data)

        if password:
            user.set_password(password)
            user.save()

        return user


class AuthTokenSerializer(serializers.Serializer):
    """Serializer for the user auth token."""

    email = serializers.EmailField()
    password = serializers.CharField(
        style={"input_type": "password"},
        trim_whitespace=False,
    )

    def validate(self, attrs):
        """Validate and authenticate the user."""
        email = attrs.get("email")
        password = attrs.get("password")
        user = authenticate(
            request=self.context.get("request"),
            username=email,
            password=password,
        )
        if not user:
            msg = _("Unable to authenticate with provided credentials.")
            raise serializers.ValidationError(msg, code="authorization")

        attrs["user"] = user
        return attrs

```


```py
# Views
"""
Views for the user API.
"""

from rest_framework import generics, authentication, permissions
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.settings import api_settings

from user.serializers import (
    UserSerializer,
    AuthTokenSerializer,
)


class CreateUserView(generics.CreateAPIView):
    """Create a new user in the system."""

    serializer_class = UserSerializer


class CreateTokenView(ObtainAuthToken):
    """Create a new auth token for user."""

    serializer_class = AuthTokenSerializer
    renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES


class ManageUserView(generics.RetrieveUpdateAPIView):
    """Manage the authenticated user."""

    serializer_class = UserSerializer
    authentication_classes = [authentication.TokenAuthentication]
    permission_classes = [permissions.IsAuthenticated]

    def get_object(self):
        """Retrieve and return the authenticated user."""
        return self.request.user

```

```py
# Urls
"""
URL mappings for the user API.
"""

from django.urls import path
from . import views

app_name = 'user'

urlpatterns = [
    path('create/', views.CreateUserView.as_view(), name='create'),
    path('token/', views.CreateTokenView.as_view(), name='token'),
    path('me/', views.ManageUserView.as_view(), name='me'),
]

```
