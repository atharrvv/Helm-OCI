# Jenkins CI/CD Pipeline for Helm OCI Chart and Docker Image

This Jenkins pipeline automates the build, packaging, and deployment of a Dockerized Python application along with its Helm chart, pushing both artifacts to Docker Hub and an OCI-compliant Helm registry.

---

## Pipeline Overview

The pipeline consists of the following key stages:

### 1. Docker Build

- Builds a Docker image `eatherv/python-application:0.0.2` from the current repository.
- Uses the Jenkins Docker plugin to build the image.

### 2. Docker Push

- Pushes the built Docker image to Docker Hub (`eatherv` namespace).
- Authenticates using Jenkins credentials.

### 3. Helm Package

- Packages the Helm chart located at `./chart/application` with version `5.0.0`.
- Generates a `.tgz` Helm chart package ready for distribution.

### 4. Helm Docker Login

- Authenticates to Docker Hub's OCI registry using Helm CLI.
- Uses Jenkins credentials for secure login.

### 5. Helm Push

- Pushes the packaged Helm chart to the OCI-compliant registry at `oci://registry-1.docker.io/eatherv`.
- This allows Helm charts to be stored and distributed via Docker Hub's OCI registry.

---

## Environment Variables

| Variable      | Description                              | Default Value                         |
|---------------|------------------------------------------|-------------------------------------|
| `CHART_NAME`  | The Helm chart name                       | `application`                       |
| `CHART_VERSION`| The Helm chart version                    | `5.0.0`                            |
| `OCI_REGISTRY`| OCI-compliant Helm chart registry URL    | `oci://registry-1.docker.io/eatherv` |

---

## Prerequisites

- Jenkins instance with Docker and Helm installed.
- Jenkins credentials set up with ID `docker` for Docker Hub access.
- Docker Hub repository and Helm OCI registry permissions.
- The Helm chart located at `./chart/application` in your repo.

---

## How to Use

1. Commit your Dockerfile and Helm chart in the repository.
2. Configure Jenkins pipeline with this Jenkinsfile.
3. Ensure Jenkins has access to Docker and Helm.
4. Trigger the pipeline to build, package, and push your Docker image and Helm chart.

---

## Benefits

- Automates container image and Helm chart lifecycle.
- Securely manages credentials via Jenkins.
- Uses Helm OCI registry support for modern chart distribution.
- Keeps versioning consistent across artifacts.

---

## Contact

For questions or support, reach out at [atharrvv](https://github.com/atharrvv).

---

Feel free to customize this pipeline to fit your deployment environment!

