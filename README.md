# Django Docker

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

# Configuring GitHub Actions
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


# Django Test
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


# Configuring DataBase
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
