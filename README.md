# VProfile Application

This repository contains the VProfile Java Spring MVC web application packaged as a WAR and designed to run behind Tomcat. It is intended to be built and deployed as part of a GitOps workflow using Maven, Docker, Amazon ECR, and a separate Helm repository.

## Overview

- Java 21
- Maven build
- Spring MVC / Spring Security application
- WAR packaging
- Tomcat deployment
- Docker multi-stage build
- GitHub Actions CI/CD pipeline
- SonarQube self-hosted scanning
- Amazon ECR image publishing
- Helm values update in a separate GitOps repo

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── ci.yml
├── Docker-files/
│   └── app/
│       └── multistage/
│           └── Dockerfile
├── src/
│   ├── main/
│   └── test/
├── pom.xml
├── sonar-project.properties
├── README.md
└── .gitignore
```

## Prerequisites

- Java 21
- Maven 3.9+
- Docker
- AWS CLI configured for ECR access
- GitHub repository with the following secrets and variables configured

### Required GitHub Secrets

- `SONAR_TOKEN`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `HELM_REPO_USER`
- `GITOPS_PAT`
- `SLACK_WEBHOOK` (if used by notifications)

### Required GitHub Variables

- `AWS_REGION` (example: `us-east-1`)
- `ECR_REPOSITORY` (example: `vprofileappimg`)
- `HELM_REPO_NAME` (example: `vprofile-helm`)
- `SONAR_HOST_URL` (self-hosted SonarQube URL)

## Build and Run Locally

### Compile and package

```bash
mvn clean package
```

### Run the application locally

This project is a Spring MVC WAR application, typically deployed to Tomcat. You can also run it with a servlet container or local app server configured for the project.

## Docker Build

The Docker image is built using the multi-stage Dockerfile at:

```text
Docker-files/app/multistage/Dockerfile
```

The pipeline builds the WAR artifact and then packages it into a Tomcat-based image.

## CI/CD Pipeline

The GitHub Actions workflow is defined in:

```text
.github/workflows/ci.yml
```

### Trigger behavior

1. Pull request to `main`
   - Runs Maven build
   - Runs unit tests
   - Runs Checkstyle
   - Runs SonarQube scan
   - Runs SonarQube Quality Gate check
   - Blocks merge if the quality gate fails

2. Push to `main`
   - Builds Docker image
   - Pushes image to Amazon ECR
   - Tags image with commit SHA and `latest`
   - Updates a Helm `values.yaml` in the separate GitOps repository

### Important behavior

- Feature branch pushes do not trigger this pipeline.
- Docker and Helm jobs do not run for pull requests.
- Docker and Helm jobs only run on push to `main`.
- ECR registry and image tag are passed between jobs using job outputs.

## SonarQube Configuration

The project uses the settings in:

```text
sonar-project.properties
```

This file defines project metadata and source/report paths used by SonarQube.

## Helm Update

The pipeline updates the image values in the separate Helm repo at:

```text
helm/vprofile/values.yaml
```

It uses `yq` to set:

```yaml
app:
  image: 
  tag: 
  replicas: 1
  containerPort: 8080
  servicePort: 8080
```

The workflow updates both `app.image` and `app.tag` fields using the ECR registry and the Git commit SHA image tag.

## Notes

- This repository is the application source repo.
- Helm configuration lives in a separate GitHub repository.
- The deployment workflow authenticates to ECR and the GitOps repository using GitHub secrets.

## License

This project is provided as-is for development and deployment workflow purposes.
