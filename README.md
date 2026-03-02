# CI/CD Project

This repository contains a comprehensive CI/CD setup for two applications: a Node.js backend and a PHP web application. Each application is containerized using Docker and includes automated build, test, and deployment pipelines using GitHub Actions.

## Project Overview

The project demonstrates modern DevOps practices with:
- **Containerization**: Both applications use Docker for consistent environments
- **Multi-stage Builds**: Optimized Docker images for production
- **Automated CI/CD**: GitHub Actions workflows for continuous integration and deployment
- **Multi-environment Deployment**: Automated deployment to VPS servers
- **Database Integration**: Prisma for Node.js and MySQL for PHP

## Project Structure

```
.
├── node/                    # Node.js application (NestJS + Prisma)
│   ├── ci/
│   │   ├── ci.matrix.yaml           # GitHub Actions workflow for Node.js (Matrix strategy)
│   │   ├── ci.vite.yaml             # GitHub Actions workflow for Vite builds
│   │   ├── notify/
│   │   │   ├── ci.notify.yaml       # Comprehensive multi-branch CI/CD pipeline
│   │   │   └── README.md            # Notify pipeline documentation
│   │   └── matrix/
│   │       └── README.md            # Matrix strategy documentation
│   ├── docker/
│   │   └── Dockerfile               # Multi-stage Docker build for Node.js
│   └── README.md                    # Node.js specific documentation
├── php/                     # PHP application (Laravel)
│   ├── bash/
│   │   └── docker-entrypoint.sh  # Docker entrypoint script
│   ├── ci/
│   │   └── ci.yaml          # GitHub Actions workflow for PHP
│   ├── docker/
│   │   ├── compose.yaml     # Docker Compose configuration
│   │   └── Dockerfile       # PHP application Dockerfile
│   └── README.md            # PHP specific documentation
├── Caddy/                   # Reverse proxy configuration
│   └── Caddyfile            # Caddy server configuration
└── README.md                # This file
```

## Components

### Node.js Application (`node/`)
- **Framework**: NestJS
- **Database**: Prisma ORM
- **Containerization**: Multi-stage Docker build with Node.js 20 Alpine
- **CI/CD**: Builds, pushes Docker image, deploys to 2 VPS servers via SSH

### PHP Application (`php/`)
- **Framework**: Laravel
- **Database**: MySQL 8.0
- **Containerization**: Docker Compose with PHP 8.2, MySQL, and queue worker
- **CI/CD**: Builds Docker image, pushes to Docker Hub, deploys to VPS via SSH
- **Automation**: Automated database migrations, seeding, and environment setup

## Prerequisites

- Docker and Docker Compose
- Git
- Node.js 20+ (for local Node.js development)
- PHP 8.2+ (for local PHP development)
- Access to Docker Hub and VPS servers for deployment

## Getting Started

### Local Development

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd CI-CD
   ```

2. **Node.js Application**:
   ```bash
   cd node
   npm install
   npm run build
   # Follow instructions in node/README.md
   ```

3. **PHP Application**:
   ```bash
   cd php/docker
   docker compose up -d --build
   # Application available at http://localhost:8000
   ```

### Docker Deployment

Each application can be run independently using Docker:

- **Node.js**: `docker build -t node-app ./node/docker`
- **PHP**: `cd php/docker && docker compose up -d`

## CI/CD Pipelines

The project includes multiple GitHub Actions workflows for different deployment scenarios:

### Node.js Pipelines

#### 1. Notify Pipeline (`node/ci/notify/ci.notify.yaml`) - **Recommended**
Comprehensive multi-branch CI/CD pipeline with advanced features:
- **Test Job**: Runs on feature branches (not `main` or `dev`)
  - Runs linting, fixes, and build tests
  - Blocks deployment if tests fail
- **Build & Push Job**: Runs on `dev` branch
  - Creates Docker image and pushes to Docker Hub
  - Enables staging environment testing
- **Deploy Job**: Runs on `main` branch
  - Deploys to production VPS
  - Pulls latest images and recreates containers
  - Restarts Caddy reverse proxy
- **Deploy Check Job**: Post-deployment verification
  - Verifies all containers are running
  - Performs HTTP health checks
  - Fails deployment if health check fails
- **Notify Job**: Failure notifications
  - Sends Telegram alerts on critical failures
  - Only notifies on `main` and `dev` branches

**For detailed documentation**, see [node/ci/notify/README.md](node/ci/notify/README.md)

#### 2. Matrix Pipeline (`node/ci/ci.matrix.yaml`)
- **Trigger**: Push to `main` branch
- **Strategy**: Matrix deployment to multiple VPS servers
- **Jobs**:
  - CI Check: Build and test
  - Build & Push: Create and push Docker image
  - Deploy: Deploy to multiple VPS servers in parallel

#### 3. Vite Pipeline (`node/ci/vite/ci.vite.yaml`)
Frontend build pipeline for Vite-based applications

### PHP Pipeline (`php/ci/ci.yaml`)
- **Trigger**: Push to `main` branch
- **Jobs**:
  - Build: Create Docker image and push to Docker Hub
  - Deploy: Deploy to VPS server via SSH

## GitHub Secrets Configuration

### For Notify Pipeline (Node.js - Recommended)
- `DOCKER_USERNAME` - Docker Hub username
- `DOCKER_PASSWORD` - Docker Hub personal access token
- `PACKAGE` - Docker repository name
- `VPS_HOST` - VPS IP address or domain
- `USERNAME` - VPS SSH username
- `VPS_ACCESS_KEY` - SSH private key for VPS access
- `PORT` - Application port for health checks
- `TELEGRAM_BOT_TOKEN` - Telegram bot token for notifications
- `TELEGRAM_CHAT_ID` - Telegram chat ID for alerts

### For Matrix & PHP Pipelines
- `DOCKER_USERNAME` & `DOCKER_PASSWORD`
- `VPS_HOST` & `VPS_PASSWORD` (or `VPS_ACCESS_KEY` for SSH key auth)
- Additional secrets for multi-server deployments

## Technologies Used

- **Containerization**: Docker, Docker Compose
- **CI/CD**: GitHub Actions
- **Node.js**: NestJS, Prisma, TypeScript
- **PHP**: Laravel, MySQL
- **Deployment**: SSH, Docker Hub

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Ensure Docker builds work
5. Push to your fork and create a pull request

For detailed setup and development instructions, see the README files in each application directory:
- [node/README.md](node/README.md) - Node.js application setup
- [php/README.md](php/README.md) - PHP application setup
- [node/ci/notify/README.md](node/ci/notify/README.md) - **Notify Pipeline Documentation** (recommended reading)