# PHP Application with CI/CD

This directory contains a PHP application configured with a complete CI/CD pipeline and Docker environment.

## Directory Structure

- **bash/**: Contains shell scripts like `docker-entrypoint.sh` for automation.
- **ci/**: Contains the CI/CD pipeline configuration (`ci.yaml`).
- **docker/**: Contains Docker configuration files (`Dockerfile`, `compose.yaml`).

## Setup & Installation

The project is containerized using Docker. To get started:

1.  **Prerequisites**: Ensure you have Docker and Docker Compose installed.
2.  **Build and Run**:
    ```bash
    cd docker
    docker compose up -d --build
    ```
3.  **Access the Application**: The application (Laravel) will be available at `http://localhost:8000`.

## Automated Setup (Entrypoint)

The project includes a `docker-entrypoint.sh` script that runs automatically when the container starts. It handles:

- **Dependency Installation**: Installs Composer and NPM dependencies if missing.
- **Environment Setup**: Creates `.env` from example and generates the app key.
- **Database Setup**: Automatically runs **migrations** (`php artisan migrate --force`) and **seeders** (`php artisan db:seed --force`) ensures the database is populated with initial data.
- **Permissions**: Sets correct permissions for storage and cache.
- **Optimization**: Caches configuration, routes, and views.

## CI/CD Pipeline

The project uses a CI/CD pipeline defined in `ci/ci.yaml` to automate building, testing, and deployment.

- **Trigger**: Pushes to the `main` branch.
- **Build**: Builds a Docker image and pushes it to Docker Hub.
- **Deploy**: Connects to a VPS via SSH and deploys the updated Docker image.

## Services

- **App**: PHP 8.2 CLI based application (Laravel).
- **MySQL**: Database service (MySQL 8.0).
- **Queue**: Laravel queue worker.
