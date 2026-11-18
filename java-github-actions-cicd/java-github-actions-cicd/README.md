# End-to-End Java CI/CD with GitHub Actions, Docker & Kubernetes

![Java](https://img.shields.io/badge/Java-17-orange?logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.5-6DB33F?logo=springboot)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=githubactions)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Deployment-326CE5?logo=kubernetes)

## Project Overview

This project demonstrates an end-to-end CI/CD workflow for a Java Spring Boot application. GitHub Actions automatically builds and tests the application, creates a Docker image, publishes the image to Docker Hub, and deploys the updated image to a Kubernetes cluster when code is pushed to the `main` branch.

> The workflow deploys to an existing Kubernetes cluster. You must configure the GitHub repository secrets and cluster access before the deployment job can run.

## CI/CD Workflow

```text
Developer pushes code to GitHub (main)
                 |
                 v
          GitHub Actions
                 |
                 v
          Build Java project
                 |
                 v
            Run tests
                 |
                 v
          Build Docker image
                 |
                 v
        Push image to Docker Hub
                 |
                 v
       Connect to Kubernetes cluster
                 |
                 v
      Update Kubernetes Deployment
                 |
                 v
       Verify rollout and application
```

## Technology Stack

- Java 17 and Spring Boot
- Maven
- JUnit 5 / Spring Boot Test
- Docker
- Docker Hub
- GitHub Actions
- Kubernetes and kubectl

## Repository Structure

```text
java-github-actions-cicd/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── src/
│   ├── main/java/com/example/demo/
│   │   └── DemoApplication.java
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java
├── .dockerignore
├── .gitignore
├── Dockerfile
└── pom.xml
```

## Run Locally

Prerequisites: JDK 17 and Maven.

```bash
mvn clean test
mvn spring-boot:run
```

Open `http://localhost:8080/` or `http://localhost:8080/health`.

## Build and Run with Docker

```bash
docker build -t java-github-actions-cicd:local .
docker run --rm -p 8080:8080 java-github-actions-cicd:local
```

## Configure GitHub Actions

### 1. Create a Docker Hub access token

Create a Docker Hub access token with permission to push images. Do not use your Docker Hub password.

### 2. Add repository secrets

In GitHub, open **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `KUBE_CONFIG` | Base64-encoded kubeconfig for your deployment cluster |

To create the base64 value for `KUBE_CONFIG`:

- Linux: `base64 -w 0 ~/.kube/config`
- macOS: `base64 < ~/.kube/config | tr -d '\n'`
- PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("$HOME\.kube\config"))`

Copy the output into the GitHub secret. Treat kubeconfig as a sensitive credential. Use a dedicated, least-privilege Kubernetes identity restricted to the `java-demo` namespace; do not use a cluster-admin credential for routine deployments.

### 3. Update the workflow image name

Open `.github/workflows/ci-cd.yml` and replace `YOUR_DOCKERHUB_USERNAME` with your Docker Hub username in the `IMAGE_NAME` value.

### 4. Prepare your Kubernetes cluster

You need an existing Kubernetes cluster whose API endpoint is reachable from GitHub-hosted runners. Ensure the kubeconfig context points to the intended cluster and the configured identity can create/update Deployments and Services and read Pods in the `java-demo` namespace.

The workflow applies the manifests, publishes the image, updates the Deployment to the commit-specific image tag, and waits for the rollout to complete.

### 5. Push to GitHub

Commit and push the project to the `main` branch. Open the repository's **Actions** tab to watch the workflow.

```bash
git add .
git commit -m "Add Java CI/CD pipeline"
git push origin main
```

## Verify Deployment

After the workflow succeeds, check your cluster:

```bash
kubectl get all -n java-demo
kubectl rollout status deployment/java-demo -n java-demo
```

The Service is a NodePort service on port `30080`. How you reach it depends on your cluster/network setup. For a local cluster, use the cluster's documented service access method or port-forward:

```bash
kubectl port-forward service/java-demo-service 8080:80 -n java-demo
```

Then open `http://localhost:8080/`.

## What the Pipeline Does

1. Checks out the source code.
2. Sets up Java 17.
3. Runs Maven tests.
4. Builds and pushes a Docker image tagged with the Git commit SHA.
5. Configures kubectl using the GitHub `KUBE_CONFIG` secret.
6. Applies Kubernetes manifests.
7. Updates the Deployment to the newly published image.
8. Waits for the Kubernetes rollout.

## Interview Explanation

"I built a CI/CD pipeline for a Java Spring Boot application using GitHub Actions. Whenever code is pushed to the main branch, the workflow checks out the source, builds the application with Maven, and runs automated tests. If the tests pass, it builds a Docker image and pushes it to Docker Hub with a commit-specific tag. The workflow then connects to an existing Kubernetes cluster, updates the Deployment to use the new image, and checks that the rollout completes successfully."

Be prepared to explain the workflow file, repository secrets, Docker image tags, Kubernetes Deployment, and how you would troubleshoot a failed test, image push, or rollout. Only describe pipeline stages you have actually run successfully.

## Security and Cost Notes

- Never commit Docker tokens, kubeconfig files, passwords, or Kubernetes secrets to Git.
- Use a dedicated Kubernetes service account or identity with namespace-scoped permissions.
- Restrict access to GitHub repository secrets and protect the `main` branch.
- This workflow assumes a reachable Kubernetes API. A private cluster needs a secure self-hosted runner or another private connectivity solution.
- Kubernetes infrastructure and Docker Hub usage may have costs or limits. Clean up lab resources when finished.

## Cleanup

```bash
kubectl delete namespace java-demo
```

## Author

**Vijay Kumar Kunja**

GitHub: [vijaycloudops](https://github.com/vijaycloudops)
