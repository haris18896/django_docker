# Django Docker

## Authenticate with Docker HUB
1. Go to [hub.docker.com](https://hub.docker.com) and login
2. Go to the `Account Settings` from your profile
3. Navigate to the `security`
4. Create new access token and add description to it
5. Save the token so that you can re-use it later I'm saving it in my slack account
6. go to the GitHub project settings => secrets === and add secrets
    * DOCKERHUB_USER
    * DOCKERHUB_TOKEN

## Using Docker with Django
1. Create a Docker file
2. choose a base image
3. install dependencies
4. setup users
5. configure docker compose
6. Define Services
    * Name
    * Mapping
    * Volume Map
7. Run all commands through docker compose `docker-compose run --rm <django-app> sh -c "python manage.py collectstatic"`

## Requirements
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


## Docker Compose
1. create a `docker-compose.yml` file in the root directory
2. after configuring run the following command `docker-compose build`
3. add `requirements.dev.txt` to the root directory, it's for local development, these aren't gonna be needed on image

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