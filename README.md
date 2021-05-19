# Before Artifact Lab

In this lab you will focus on the binaries and supporting systems that produce a final versioned artifact.

## Steps

### Step 1 - Hadolint (Dockerfile Lint)

1. Hadolint - Provides static analysis on Dockerfiles to ensure best practices are used from the Docker community

1. Install `hadolint`

    ```bash
    export HADOLINT_VERSION="2.4.1"
    mkdir -p /tmp/hadolint
    wget -O /tmp/hadolint/hadolint https://github.com/hadolint/hadolint/releases/download/v${HADOLINT_VERSION}/hadolint-Linux-x86_64
    chmod +x /tmp/hadolint/hadolint
    mv /tmp/hadolint/hadolint $(go env GOPATH)/bin/hadolint
    ```

1. Run `hadolint` against the Dockerfile with the provided policy file

    ```bash
    hadolint --config policies/hadolint.yaml Dockerfile
    ```

    > NOTE: The command will fail because the `Dockerfile` has a reference to a relative path `WORKDIR` directive

1. Fix the `Dockerfile` by removing the offending line and uncommenting the proper line (hint: line 18 ish)

1. Re-run the `hadolint` command against the updated `Dockerfile`

    ```bash
    hadolint --config policies/hadolint.yaml Dockerfile
    ```

    > SUCCESS!

1. In your CI/CD pipeline, this should be run within the Source Code stage.

1. Ready to move on to Step 2

### Step 2 - Container Structure Tests

In this section of the lab, we will build two docker images, one that uses Ubuntu as the final artifact, and one that uses a Distroless base image. The resulting Docker image in the Distroless will not have SSH, no extranious binaries/libraries and will not contain Sources/Update capabilities thus resulting in a smaller surface attack vector. This section will show how Container Structure Tests can prove prove that a final artifact does not have these types surface attack vectors.

1. Container structure tests - Uses static code analysis to determine if the constructed docker image is designed to the required specification.

1. Install `container-structure-test` project

    ```bash
    mkdir -p /tmp/container-structure-test
    wget -O /tmp/container-structure-test/container-structure-test-linux-amd64 https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64
    chmod +x /tmp/container-structure-test/container-structure-test-linux-amd64
    mv /tmp/container-structure-test/container-structure-test-linux-amd64 $(go env GOPATH)/bin/container-structure-test
    ```

1. Build two docker containers locally

    ```bash
    docker build -f Dockerfile-distroless -t distroless .
    docker build -f Dockerfile-ubuntu -t ubuntu .
    ```

1. Run `container-structure-test` against the `Dockerfile-ubuntu`

    ```bash
    container-structure-test test --image ubuntu:latest --config policies/container-structure-policy.yaml
    ```

    > NOTE: This should fail due to the policy indicating that SSH access should be disabled along with `sources`

    ```
    ========================================================
    ====== Test file: container-structure-policy.yaml ======
    ========================================================
    === RUN: File Existence Test: Root folder is executable
    --- PASS
    duration: 0s
    === RUN: File Existence Test: Application binary exists
    --- PASS
    duration: 0s
    === RUN: File Existence Test: Debian Sources do NOT exist
    --- FAIL
    duration: 0s
    Error: File /etc/apt/sources.list should not exist but does
    === RUN: File Existence Test: No Bash shell is available
    --- FAIL
    duration: 0s
    Error: File /bin/bash should not exist but does
    === RUN: File Existence Test: No SSH
    --- PASS
    duration: 0s

    =========================================================
    ======================== RESULTS ========================
    =========================================================
    Passes:      3
    Failures:    2
    Duration:    0s
    Total tests: 5

    FAIL
    FATA[0021] FAIL
    ```

1. Run `container-structure-test` against the `Dockerfile-distroless`

    ```bash
    container-structure-test test --image distroless:latest --config policies/container-structure-policy.yaml
    ```

    > SUCCESS!!

    ```
    ========================================================
    ====== Test file: container-structure-policy.yaml ======
    ========================================================
    === RUN: File Existence Test: Root folder is executable
    --- PASS
    duration: 0s
    === RUN: File Existence Test: Application binary exists
    --- PASS
    duration: 0s
    === RUN: File Existence Test: Debian Sources do NOT exist
    --- PASS
    duration: 0s
    === RUN: File Existence Test: No Bash shell is available
    --- PASS
    duration: 0s
    === RUN: File Existence Test: No SSH
    --- PASS
    duration: 0s

    =========================================================
    ======================== RESULTS ========================
    =========================================================
    Passes:      5
    Failures:    0
    Duration:    0s
    Total tests: 5

    PASS
    ```
