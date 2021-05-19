# Before Artifact Lab

In this lab you will focus on the binaries and supporting systems that produce a final versioned artifact.

## Steps

### Hadolint (Dockerfile Lint)

1. Hadolint - Provides static analysis on Dockerfiles to ensure best practices are used from the Docker community

1. Install `hadolint`

    ```bash
    export HADOLINT_VERSION="2.4.1"
    mkdir -p /tmp/hadolint
    wget -O /tmp/hadolint/hadolint https://github.com/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-Linux-x86_64
    chmod +x /tmp/hadolint/hadolint
    mv /tmp/hadolint/hadolint $(go env GOPATH)/bin/hadolint
    ```

1. Run `hadolint` against the Dockerfile

### Container Structure Tests

1. Container structure tests - Uses static code analysis to determine if the constructed docker image is designed to the required specification

1. Install `container-structure-test` project

    ```bash
    mkdir -p /tmp/container-structure-test
    wget -O /tmp/container-structure-test/container-structure-test-linux-amd64 https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64
    chmod +x /tmp/container-structure-test/container-structure-test-linux-amd64
    mv /tmp/container-structure-test/container-structure-test $(go env GOPATH)/bin/container-structure-test
    ```

1. Run `hadolint` against the Dockerfile

    ```bash
    hadolint --config security/hadolint.yaml Dockerfile
    ```

1. Build the image and push to the local Container Repository (GCR)

    ```bash
    gcloud builds submit --tag gcr.io/${PROJECT_ID}/hello-world
    ```

1. Run `container-structure-test`
    ```bash
    container-structure-test test --image "${IMAGE}:${TAG}" --config policies/container-structure-policy.yaml
    ```

