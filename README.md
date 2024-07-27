# Wisecow Deployment

## Overview

This repository contains the configuration and deployment setup for the Wisecow application. It integrates Docker for containerization, GitHub Actions for CI/CD, and Kubernetes for orchestration. This guide covers the setup of these components and the use of `act` for local testing.

## Prerequisites

* **Docker**: Install Docker from [Docker's official website](https://www.docker.com/get-started).
* **Kubernetes Cluster**: You can use Minikube for local testing or any remote Kubernetes cluster.
* **GitHub Actions**: Set up for CI/CD workflows.
* **act**: Tool for running GitHub Actions locally. Install it from [act's GitHub page](https://github.com/nektos/act).

## Repository Structure

* `Dockerfile`: Defines the Docker image for the Wisecow application.
* `wisecow-deployment.yaml`: Kubernetes deployment configuration.
* `wisecow-service.yaml`: Kubernetes service configuration.
* `.github/workflows/deploy.yml`: GitHub Actions workflow for CI/CD.
* `.env`: Environment file for local testing with `act`.

## Docker Setup

### Build the Docker Image

Ensure that you have a valid `Dockerfile` in the root directory of your repository. To build the Docker image, use the following command:

```bash
docker build -t your-dockerhub-username/wisecow:latest .
```

Push the Docker Image
Once the image is built, push it to DockerHub:

```bash
docker push your-dockerhub-username/wisecow:latest
```

Replace your-dockerhub-username with your DockerHub username.

Kubernetes Configuration
Deployment YAML
Create or update wisecow-deployment.yaml to define the deployment of the Wisecow application. Here is a sample:

```kubernetes

yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: your-dockerhub-username/wisecow:latest
        ports:
        - containerPort: 4499

```

Service YAML
Create or update wisecow-service.yaml to define the service configuration. Example:

```kubernetes
yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
  - protocol: TCP
    port: 80
    targetPort: 4499
  type: LoadBalancer

```

GitHub Actions
Workflow Configuration
Create or update the workflow file .github/workflows/deploy.yml:

```gh.actions
yaml
Copy code
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up kubectl
        uses: azure/setup-kubectl@v1
        with:
          version: 'v1.18.0'

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f wisecow-deployment.yaml
          kubectl apply -f wisecow-service.yaml
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}
```
Secrets
Add the following secrets in GitHub repository settings:

KUBECONFIG: Base64 encoded kubeconfig file content for Kubernetes access.
Local Testing with act
Create .env File
Create a .env file in the root directory for local testing with act. This file should contain environment variables and secrets:

```env
DOCKER_USERNAME=your_dockerhub_username
DOCKER_PASSWORD=your_dockerhub_password
KUBECONFIG=/path/to/your/kubeconfig
```

Run act
To test the GitHub Actions workflow locally, use the act command:

```terminal
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:full-latest --env-file .env
```

Troubleshooting
ImagePullBackOff: If you encounter ImagePullBackOff, ensure that the Docker image name and tag are correct and that the image is available in DockerHub.
Kubernetes Deployments: Check Kubernetes pod logs if the deployment fails or is stuck. Use kubectl logs <pod-name> to view the logs.
Secrets Issues: Ensure that secrets in GitHub are correctly configured and that .env file used for local testing does not contain invalid characters.


License
This project is licensed under the MIT License - see the LICENSE file for details.


