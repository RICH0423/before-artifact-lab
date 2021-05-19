# Before Artifact Lab

In this lab you will focus on the binaries and supporting systems that produce a final versioned artifact.

## Steps

### Container Structure Tests

1. Container structure tests - Uses static code analysis to determine if the constructed docker image is designed to the required specification

1. Install `container-structure-test` project

    ```bash
    mkdir -p /tmp/container-structure-test
    wget -O /tmp/container-structure-test/container-structure-test-linux-amd64 https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64
    chmod +x /tmp/container-structure-test/container-structure-test-linux-amd64
    mv /tmp/container-structure-test/container-structure-test $(go env GOPATH)/bin/container-structure-test
    ```

1. Build the image and push to the local Container Repository (GCR)

    ```bash
    gcloud builds submit
    ```

1. Run `container-structure-test`
    ```bash
    container-structure-test test --image "${IMAGE}:${TAG}" --config policies/container-structure-policy.yaml
    ```

