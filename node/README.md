# Node.js Application

This is a Node.js application built with NestJS and Prisma, containerized using Docker for easy deployment.

## Prerequisites

- Node.js 20 or later
- Docker and Docker Compose
- npm or yarn

## Local Development

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd node
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up the database (assuming Prisma is configured):
   ```bash
   npx prisma generate
   npx prisma db push
   ```

4. Build the application:
   ```bash
   npm run build
   ```

5. Run the application:
   ```bash
   npm start
   ```

## Docker

The application is containerized using a multi-stage Docker build for optimized production images.

### Build the Docker Image

```bash
docker build -t your-app-name ./docker
```

### Run with Docker Compose

Assuming you have a `docker-compose.yml` file in the project root or appropriate location:

```bash
docker compose up -d
```

The Dockerfile uses Node.js 20 Alpine as the base image, includes Prisma client generation, and runs as a non-root user for security.

## CI/CD

The project uses GitHub Actions for continuous integration and deployment. The workflow (`ci/ci.matrix.yaml`) includes:

- **CI Check**: Builds and tests the application on pushes to the main branch.
- **Build & Push Docker Image**: Builds and pushes the Docker image to Docker Hub when CI passes.
- **Deploy**: Deploys to two VPS servers using SSH and Docker Compose.

### Required Secrets for GitHub Actions

Set the following secrets in your GitHub repository:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password
- `VPS1_HOST`: Hostname/IP of the first VPS
- `VPS1_PASSWORD`: SSH password for the first VPS
- `VPS2_HOST`: Hostname/IP of the second VPS
- `VPS2_PASSWORD`: SSH password for the second VPS

### Deployment Process

On push to the `main` branch:
1. Code is checked out and Node.js is set up.
2. Dependencies are installed and the project is built.
3. If build succeeds, a Docker image is built and pushed to Docker Hub.
4. The image is then pulled and deployed to both VPS servers using Docker Compose with force recreate and orphan removal.

## Project Structure

- `docker/Dockerfile`: Multi-stage Docker build configuration
- `ci/ci.matrix.yaml`: GitHub Actions workflow for CI/CD
- `prisma/`: Database schema and migrations
- `src/`: Application source code
- `dist/`: Built application (generated)

## Technologies Used

- **NestJS**: Node.js framework
- **Prisma**: Database ORM
- **Docker**: Containerization
- **GitHub Actions**: CI/CD pipeline</content>
<parameter name="filePath">/Users/betopia/Desktop/Arafat/credentials/CI-CD/node/README.md