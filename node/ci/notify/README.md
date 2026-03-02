# CI/CD Pipeline Documentation

## Overview

This GitHub Actions workflow automates testing, building, deploying, and monitoring of a Node.js application across different environments. The pipeline is triggered on every push to any branch and adapts its behavior based on which branch the code is being pushed to.

## Pipeline Structure

The pipeline consists of **5 main jobs** that execute based on branch triggers:

### 1. **Test Job** (Branches other than `main` and `dev`)
**Trigger:** Push to any branch except `main` and `dev`

**Purpose:** Validates code quality and builds the application in a controlled test environment.

**Steps:**
- Checkout the code
- Setup Node.js v24
- Install dependencies with `npm install --force`
- Run linting and fixes with `npm run ci:fix`
- Build the application with `npm run build`

**Status:** If any step fails, the pipeline stops and notifies developers.

---

### 2. **Build & Push Job** (Branch: `dev`)
**Trigger:** Push to `dev` branch only

**Purpose:** Builds a Docker image and pushes it to Docker Hub for staging/testing.

**Steps:**
- Checkout the code
- Authenticate to Docker Hub using stored credentials
- Setup Docker environment
- Build Docker image with tag: `{DOCKER_USERNAME}/{PACKAGE}:latest`
- Push the image to Docker Hub repository

**Requirements:**
- `DOCKER_USERNAME` secret
- `DOCKER_PASSWORD` secret
- `PACKAGE` secret (Docker Hub repository name)

**Note:** This job skips when pushing to `main` or other branches.

---

### 3. **Deploy Job** (Branch: `main`)
**Trigger:** Push to `main` branch only

**Purpose:** Deploys the application to production VPS using Docker Compose.

**Steps:**
1. Checkout the code
2. Verify SSH connection to VPS
3. Copy `docker-compose.yaml` to the VPS project directory
4. Pull latest images and redeploy containers:
   - Pull the latest Docker images
   - Stop running containers
   - Clean up unused images
   - Start containers with forced recreation
5. Restart Caddy reverse proxy server

**Requirements:**
- `VPS_HOST` secret (VPS IP or domain)
- `USERNAME` secret (VPS user)
- `VPS_ACCESS_KEY` secret (SSH private key)
- `PACKAGE` secret (project directory name)

---

### 4. **Deploy Check Job** (Branch: `main`)
**Trigger:** Push to `main` branch (runs after Deploy job completes)

**Purpose:** Verifies that the deployment was successful and the application is healthy.

**Checks:**
1. **Container Status:** Ensures all Docker containers are running
   - Queries container states
   - Fails if any container is exited or restarting
2. **Application Health:** Performs an HTTP health check
   - Sends request to `http://localhost:{PORT}/api/v1`
   - Expects HTTP 200 response
   - Fails if health check returns any other status code

**Requirements:**
- `PORT` secret (application port)
- Successful completion of Deploy job

**Failure Behavior:** If containers aren't running or health check fails, the job exits with code 1, triggering the notification system.

---

### 5. **Notify Job** (Failure Notifications)
**Trigger:** Runs always, notifies on failure in `main` or `dev` branches

**Purpose:** Sends alerts when any critical job fails in production or staging deployments.

**Conditions:**
- Only runs if at least one of these jobs fails:
  - Build & Push job
  - Deploy job
  - Deploy Check job
- Only sends notification if on `main` or `dev` branch

**Action:** Sends Telegram message with:
- Alert emoji (🚨)
- Failure indicator
- Instructions to check GitHub Actions logs

**Requirements:**
- `TELEGRAM_BOT_TOKEN` secret
- `TELEGRAM_CHAT_ID` secret

---

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Push to Repository                            │
└───────────────────────┬─────────────────────────────────────────┘
                        │
            ┌───────────┼───────────┐
            │           │           │
            ▼           ▼           ▼
        ┌──────┐   ┌──────────┐   ┌───────┐
        │ Test │   │Build&Push│   │Deploy │
        │ Job  │   │  Job(dev)│   │Job(main)
        └──────┘   └──────────┘   └───┬───┘
           │           │              │
           │           │              ▼
           │           │          ┌──────────┐
           │           │          │Deploy    │
           │           │          │Check Job │
           │           │          └────┬─────┘
           │           │               │
           └───────────┴───────────────┘
                       │
                       ▼
              ┌────────────────┐
              │Notify on       │
              │Failure (Telegram)
              └────────────────┘
```

---

## Branch Strategy

| Branch | Behavior |
|--------|----------|
| `main` | Triggers: Deploy → Deploy Check → Notify (if failed) |
| `dev` | Triggers: Build & Push → Notify (if failed) |
| `feature/*`, `bugfix/*`, etc. | Triggers: Test only |

---

## Environment Secrets

All sensitive information must be configured as GitHub repository secrets:

| Secret | Purpose | Example |
|--------|---------|---------|
| `DOCKER_USERNAME` | Docker Hub username | `myusername` |
| `DOCKER_PASSWORD` | Docker Hub personal access token | `dckr_pat_...` |
| `PACKAGE` | Docker repository name | `my-app` |
| `VPS_HOST` | VPS IP address or domain | `192.168.1.100` |
| `USERNAME` | VPS SSH username | `deploy` |
| `VPS_ACCESS_KEY` | SSH private key for VPS access | Multiline SSH key |
| `PORT` | Application port for health check | `3000` |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token for notifications | `123456789:ABCDefGHIjklmnoPQRstuvWXYZ` |
| `TELEGRAM_CHAT_ID` | Telegram chat ID for notifications | `987654321` |

---

## Usage Guidelines

### For Feature Development
1. Create a feature branch from `dev`
2. Push changes → Automatic test runs
3. Fix any test failures
4. Create Pull Request to `dev`

### For Staging Deployment
1. Merge PR into `dev` branch
2. Automatic build and Docker image push to Hub
3. Manual deployment to staging environment (if configured)

### For Production Deployment
1. Merge `dev` → `main` (via PR)
2. Automatic deployment to production VPS
3. Automatic health checks
4. Telegram notification on failure

---

## Error Handling

- **Test Failures:** Blocks progression, developer must fix
- **Build Failures:** Blocks deployment, Docker image not created
- **Deploy Failures:** Caddy restart included, check VPS logs
- **Health Check Failures:** Automatic notification sent, manual intervention may be required
- **All Critical Failures:** Telegram alert sent to team

---

## Monitoring & Logs

- View all job logs in GitHub Actions
- Check VPS deployment logs: `docker compose logs -f`
- Monitor Telegram channel for failure alerts
- Health check endpoint: `http://localhost:{PORT}/api/v1`

---

## Security Notes

- All secrets are encrypted and never logged
- SSH key access limited to specific VPS operations
- Docker credentials only used during build/push phase
- Telegram token only used for notifications
- All sensitive data should follow principle of least privilege

