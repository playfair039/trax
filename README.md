# CI Load Test Assignment

## Introduction

This project implements a Continuous Integration (CI) workflow that automates the deployment and load testing of HTTP services in a Kubernetes cluster. The workflow is triggered by pull requests to the `main` branch and performs the following tasks:

- Provisions a multi-node Kubernetes cluster using KinD (Kubernetes in Docker).
- Deploys an NGINX Ingress Controller to manage incoming HTTP requests.
- Creates two `http-echo` deployments serving responses "foo" and "bar".
- Configures ingress routing to direct traffic based on hostnames `foo.localhost` and `bar.localhost`.
- Ensures the health of the deployments and ingress.
- Performs load testing using `hey` and posts the results as a comment on the pull request.

## Time Taken

Approximately **4 hours** were spent on this assignment, including setting up the environment, writing the workflow, testing, and documenting the implementation.

## Implementation Details

### CI Workflow with GitHub Actions

**Why GitHub Actions?**

- **Integration**: Seamless integration with GitHub repositories.
- **Ease of Use**: Simple syntax and a wide range of pre-built actions.
- **Security**: Managed secrets and fine-grained permissions.
- **Scalability**: Ability to run workflows on different runners.

### Provisioning the Kubernetes Cluster

**Tool Used**: **KinD (Kubernetes in Docker)**

**Why KinD?**

- **Lightweight**: Ideal for CI environments with limited resources.
- **Simplicity**: Easy to set up and tear down clusters.
- **Docker-Based**: No need for nested virtualization in CI runners.
- **Multi-Node Support**: Can simulate a real cluster with multiple nodes.

**Cluster Configuration:**

- **Multi-Node**: Configured with one control plane and two worker nodes.
- **Port Mappings**: Mapped container ports 80 and 443 to host ports 8080 and 8443 to avoid permission issues.
- **Node Labels**: Labeled the control-plane node with `ingress-ready=true` to schedule the ingress controller properly.

### Deploying NGINX Ingress Controller

**Version Used**: **v1.9.1**

**Why This Version?**

- **Compatibility**: Ensures compatibility with the Kubernetes version used by KinD.
- **Stability**: Provides the latest stable features without breaking changes.
- **Community Support**: Widely used and well-documented.

### Deploying HTTP Echo Services

**Services Deployed**:

- **http-echo-foo**: Returns "foo" in the response.
- **http-echo-bar**: Returns "bar" in the response.

**Why HTTP Echo?**

- **Simplicity**: Lightweight and minimal configuration.
- **Purpose-Built**: Ideal for testing HTTP responses without complex application logic.

### Configuring Ingress Routing

**Ingress Resources**:

- Defined separate ingress resources for `foo.localhost` and `bar.localhost`.
- Specified `ingressClassName: nginx` to ensure the NGINX Ingress Controller processes the ingress resources.
--
- Port mapping required given lack of access to ports 80 and 443 within the environment

**Why Separate Ingresses?**

- **Clarity**: Easier to manage and troubleshoot individual ingress rules.
- **Scalability**: Allows for independent modifications in the future.

### Ensuring Health Before Proceeding

**Health Checks**:

- Used `kubectl wait` commands to ensure deployments and ingress resources are ready.
- Added a `sleep` before testing connectivity to allow services to initialize fully.

**Why This Approach?**

- **Reliability**: Prevents the workflow from proceeding if components are not ready.
- **Error Handling**: Fails fast and provides clear error messages for debugging.

### Load Testing with Hey

**Tool Used**: **hey**

**Why Hey?**

- **Lightweight**: Minimal overhead on the CI runner.
- **Simple Syntax**: Easy to configure and automate.
- **Performance Metrics**: Provides detailed statistics on request latency and throughput.

**Load Testing Parameters**:

- **Duration (`-z 30s`)**: Runs the test for 30 seconds.
- **Query Rate (`-q 10`)**: Sends 10 queries per second.

### Posting Results to Pull Request

**Method Used**: **actions/github-script@v6**

**Why actions/github-script?**

- **No Additional Installations**: Avoids installing the GitHub CLI (`gh`).
- **Flexibility**: Allows running JavaScript code to interact with GitHub's API.
- **Simplicity**: Reduces complexity in the workflow file.

**Implementation Details**:

- Read the parsed results from load testing.
- Constructed a markdown-formatted comment.
- Posted the comment to the pull request using GitHub's REST API.

### Clean-Up

**KinD Cluster Deletion**:

- Ensured the cluster is deleted regardless of the workflow's success or failure using `if: always()`.
- **Why?**: Prevents resource leakage and keeps the CI environment clean for subsequent runs.

## Choices and Justifications

### Declarative Over Imperative

- **Kubernetes Manifests**: Used declarative YAML files within `kubectl apply` commands.
- **Why?**: Ensures idempotent operations and easier maintenance.

### Avoiding Boilerplate

- **Combined Resources**: Where possible, combined multiple Kubernetes resources in a single `kubectl apply` to reduce repetition.
- **Functions in Shell Scripts**: Used functions like `parse_results` to avoid code duplication.

### Tool Selection

- **KinD**: Preferred over Minikube due to its simplicity and compatibility with container-based environments.
- **NGINX Ingress Controller**: Chosen for its widespread use and robust feature set.
- **Hey**: Selected for its ease of use in generating HTTP load and capturing essential metrics.

### Version Pinning

- **Consistent Environments**: Specified exact versions of tools (e.g., KinD, NGINX Ingress Controller) to prevent unexpected behavior due to updates.
- **Why?**: Enhances reproducibility and stability of the CI workflow.

### Security Considerations

- **Non-Privileged Ports**: Mapped container ports to host ports above 1024 to avoid permission issues in the CI environment.
- **Least Privilege**: Avoided running commands with unnecessary elevated privileges.

## How to Run the Workflow

1. **Triggering the Workflow**: Create a pull request to the `main` branch of the repository.
2. **Workflow Execution**: GitHub Actions will automatically start the CI workflow defined in `.github/workflows/ci-load-test.yaml`.
3. **Monitoring Progress**: Navigate to the "Actions" tab in GitHub to view the workflow's progress and logs.
4. **Viewing Load Test Results**: Once the workflow completes, a comment with the load testing results will be posted on the pull request.

## Potential Improvements

- **Parameterization**: Introduce variables to make the workflow more flexible for different environments.
- **Error Handling**: Implement retries or more sophisticated error checking for network-related tasks.
- **Monitoring Solution**: As a stretch goal, integrate Prometheus to capture resource utilisation and enhance the load testing report.

## Conclusion

This implementation provides a robust CI workflow that automates the deployment and testing of services in a Kubernetes cluster. By making careful technology choices and adhering to best practices, the workflow is maintainable, efficient, and effective in achieving the desired outcomes.
