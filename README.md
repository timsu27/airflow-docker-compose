# Apache Airflow running on Docker Compose

## Building custom images with extra provider packages

Airflow provides several convenient ways to build your custom images. Here are steps that I use most of the time.

1. clone airflow repo then `cd` into airflow repo.

    ```bash
    # /bin/bash
    git clone https://github.com/apache/airflow.git
    cd airflow
    ```

2. choose python version & airflow version you intend to, then export as an environment variable. In this case, use python 3.8 & airflow 2.6.2.

    ```bash
    export DOCKER_BUILDKIT=1
    export PYTHON_VERSION=3.8
    export AIRFLOW_VERSION=2.6.2
    ```

3. build image

    ```bash
    docker build . \
        --pull \
        --build-arg PYTHON_BASE_IMAGE="python:${PYTHON_VERSION}-slim-bullseye" \
        --build-arg AIRFLOW_VERSION="${AIRFLOW_VERSION}" \
        --tag "my-registry/airflow:${AIRFLOW_VERSION}-python${PYTHON_VERSION}" \
    ```

4. if extra provider packages are needed, add `ADDITIONAL_AIRFLOW_EXTRAS` as build arguments.

    ```bash
    docker build . \
        --pull \
        --build-arg PYTHON_BASE_IMAGE="python:${PYTHON_VERSION}-slim-bullseye" \
        --build-arg AIRFLOW_VERSION="${AIRFLOW_VERSION}" \
        --tag "my-registry/airflow:${AIRFLOW_VERSION}-python${PYTHON_VERSION}" \
        --build-arg ADDITIONAL_AIRFLOW_EXTRAS="ssh,google,docker" \
    ```

## Running apache airflow 2.x on docker compose with celery executor with local development purpose

Here are the steps to take to get airflow running on docker compose on your machine.
Assume Docker Compose is installed.
The `docker-compose.override.yaml` contains the environment variable for development use. the compose file binds local directories `dags`, `configs`, `logs`, `plugins` into airflow container.

1. Launch airflow with docker compose.

    ```bash
    docker compose up -d
    ```

1. Check the running containers are running healthy.

    ```bash
    docker compose ps
    ```

1. Open browser and type http://0.0.0.0:8080 to launch the airflow webserver

1. develop & test your dag files under `dags` folder.

## Running airflow with production purpose

For production purposes, version control of dags is required. To do so, we use `git-sync` as a side-car to manage dags between versions.

1. create `.env` file containing keys inside `.env.example`. `FERNET_KEY` is the key to encrypting your connection secrets or variable secrets. `GIT_USERNAME` & `GIT_SECRET` is the git username and git credentials. `_AIRFLOW_WWW_USER_USERNAME` & `_AIRFLOW_WWW_USER_PASSWORD` is the admin of webserver. Keep these key in your production environment and never leak them.

2. Launch airflow

Use `docker-compose.prod.yaml` instead of `docker-compose.override.yaml`.

```bash
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml up -d
```

3. Go to <http://0.0.0.0:8080> and login airflow webserver with the user assigned as `_AIRFLOW_WWW_USER_USERNAME`.
