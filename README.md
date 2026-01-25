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
│   │   └── ci.matrix.yaml   # GitHub Actions workflow for Node.js
│   ├── docker/
│   │   └── Dockerfile       # Multi-stage Docker build for Node.js
│   └── README.md            # Node.js specific documentation
├── php/                     # PHP application (Laravel)
│   ├── bash/
│   │   └── docker-entrypoint.sh  # Docker entrypoint script
│   ├── ci/
│   │   └── ci.yaml          # GitHub Actions workflow for PHP
│   ├── docker/
│   │   ├── compose.yaml     # Docker Compose configuration
│   │   └── Dockerfile       # PHP application Dockerfile
│   └── README.md            # PHP specific documentation
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

Both applications use GitHub Actions for automated CI/CD:

### Node.js Pipeline (`node/ci/ci.matrix.yaml`)
- **Trigger**: Push to `main` branch
- **Jobs**:
  - CI Check: Build and test
  - Build & Push: Create and push Docker image
  - Deploy: Deploy to 2 VPS servers using matrix strategy

### PHP Pipeline (`php/ci/ci.yaml`)
- **Trigger**: Push to `main` branch
- **Jobs**:
  - Build: Create Docker image and push to Docker Hub
  - Deploy: Deploy to VPS server via SSH

### Required Secrets
Set these in your GitHub repository settings:
- `DOCKER_USERNAME` & `DOCKER_PASSWORD`
- `VPS_HOST` & `VPS_PASSWORD` (or multiple for Node.js matrix deployment)

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

For detailed setup and development instructions, see the README files in each application directory (`node/README.md` and `php/README.md`).