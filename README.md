# Lab Assignment 5

## Title
Docker CI Integration and Docker Hub Push using GitHub Actions

## Aim
Integrate Docker with a GitHub Actions CI pipeline and automate building and pushing Docker images to Docker Hub.

## Objectives
- Understand Docker and GitHub Actions integration.
- Automate Docker image building using CI.
- Push Docker images securely to Docker Hub.
- Implement CI best practices using GitHub Actions.

## Problem Statement
Manual Docker image building and deployment are time-consuming and error-prone. An automated CI pipeline is needed to build Docker images on every code change and publish them to Docker Hub.

Docker is a containerization platform that packages applications and dependencies into lightweight containers. GitHub Actions is a CI/CD service that automates workflows such as build, test, and deployment. Integrating Docker with GitHub Actions improves delivery speed and reliability through automatic build and push.

---

## Part 1: Docker with GitHub Actions CI Pipeline

### Step 0: Prerequisites
Make sure you have:
- Docker installed
- Git installed
- GitHub account
- Docker Hub account

Verify installations:

```bash
docker --version
git --version
```

### Step 1: Create Project Directory
```bash
mkdir docker-ci-demo
cd docker-ci-demo
```

Explanation:
- `mkdir docker-ci-demo`: Creates a new project folder.
- `cd docker-ci-demo`: Enters the project directory.

### Step 2: Create Application File
```bash
mkdir app
nano app/index.html
```

Add the following HTML:

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Docker CI Integration</h1>
  </body>
</html>
```

Explanation:
- `mkdir app`: Creates folder for application files.
- `nano app/index.html`: Creates and edits HTML file.
- This file will be served by Nginx inside Docker.

### Step 3: Create Dockerfile
```bash
nano Dockerfile
```

Add:

```dockerfile
FROM nginx:latest
COPY app /usr/share/nginx/html
EXPOSE 80
```

Explanation:
- `FROM nginx:latest`: Uses Nginx as the base image.
- `COPY app /usr/share/nginx/html`: Copies website files.
- `EXPOSE 80`: Exposes port 80 inside the container.

### Step 4: Test Docker Locally
Before CI, always test locally.

Build image:

```bash
docker build -t docker-ci-demo .
```

Run container:

```bash
docker run -d -p 8080:80 docker-ci-demo
```

What it does:
- `-d`: Runs container in background.
- `-p 8080:80`: Maps local port `8080` to container port `80`.

Open in browser:

```text
http://localhost:8080
```

If it works, CI should work. If it fails, your pipeline will fail.

Stop container:

```bash
docker ps
docker stop <container_id>
```

Replace `<container_id>` with the ID shown by `docker ps`.

### Step 5: Initialize Git Repository
```bash
git init
git add .
git commit -m "Initial commit with Dockerfile"
```

Explanation:
- `git init`: Initializes Git repository.
- `git add .`: Adds all files.
- `git commit`: Saves project state.

### Step 6: Create GitHub Repository
1. Go to GitHub.
2. Click **New Repository**.
3. Name it `docker-ci-demo`.
4. Do **not** initialize with README.
5. Click **Create**.

### Step 7: Connect Local Repo to GitHub
```bash
git branch -M main
git remote add origin https://github.com/<username>/docker-ci-demo.git
git push -u origin main
```

Replace `<username>` with your GitHub username.

Ubuntu/Git authentication note (important):
- While pushing over HTTPS, Git asks for `Username` and `Password`.
- Use your GitHub username as `Username`.
- Do not use your GitHub account password as `Password`.
- Create a token from GitHub:
  1. GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)
  2. Click **Generate new token (classic)**
  3. Select scopes: `repo` and `workflow`
  4. Copy the token and use it as the Git `Password` when prompted

Explanation:
- `git branch -M main`: Renames branch to `main`.
- `git remote add origin`: Connects local repo to GitHub.
- `git push -u origin main`: Uploads code.

### Step 8: Create Workflow Directory
```bash
mkdir -p .github/workflows
nano .github/workflows/docker-ci.yml
```

### Step 9: Add CI Workflow File
```yaml
name: Docker CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker Image
        run: docker build -t docker-ci-demo:latest .
```

Explanation:
- `on: push`: Triggers workflow on push.
- `runs-on: ubuntu-latest`: Uses Ubuntu runner.
- `actions/checkout`: Downloads repository code.
- `docker build`: Builds Docker image.

### Step 10: Push Workflow
```bash
git add .
git commit -m "Added Docker CI workflow"
git push origin main
```

### Step 11: Verify CI
Go to GitHub -> **Actions** tab -> check workflow logs.

---

## Part 2: Push Image to Docker Hub

### Step 1: Create Docker Hub Repository
1. Log in to Docker Hub.
2. Click **Create Repository**.
3. Name it `docker-ci-demo`.
4. Set visibility to **Public**.
5. Click **Create**.

### Step 2: Generate Docker Hub Access Token
1. Docker Hub -> **Account Settings**.
2. Open **Personal access tokens**.
3. Click **New Access Token**.
4. Copy the token.

### Step 3: Add GitHub Secrets
Go to:
GitHub -> Repository -> Settings -> Secrets and variables -> Actions -> **New repository secret**

Add these secrets:

| Secret Name       | Value                         |
|-------------------|-------------------------------|
| `DOCKER_USERNAME` | `<your_dockerhub_username>`   |
| `DOCKER_PASSWORD` | `<your_dockerhub_access_token>` |

### Step 4: Update Workflow for Push
Edit workflow file:

```bash
nano .github/workflows/docker-ci.yml
```

Replace with:

```yaml
name: Docker CI with Push

on:
  push:
    branches: ["main"]

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/docker-ci-demo:latest
```

### Step 5: Commit and Push
```bash
git add .github/workflows/docker-ci.yml
git commit -m "Added Docker Hub push workflow"
git push origin main
```

### Step 6: Verify on GitHub Actions
Go to GitHub -> **Actions** -> open workflow.

Check for:
- Docker login success
- Build success
- Push success

### Step 7: Verify on Docker Hub
Log in to Docker Hub and open repository. You should see:
- `latest` tag
- Recent push timestamp
