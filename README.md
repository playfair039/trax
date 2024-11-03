### CI Load Task - Implementation notes


**General notes**:

- KinD `v0.20.0` is deployed, it's default kubernetes version `1.28.0` aligns with Nginx ingress controller `v1.9.1`
- Container ports `80 and 443` mapped to host ports `8080 and 8443` to avoid permission issues within the execution env
- Used `kubectl wait` commands to ensure deployments and ingress resources are ready
- Added a `sleep` before testing connectivity to allow services to initialize fully to improve reliability and flakiness
- Basic 30-second load testing at 10 requests ps using `hey` writing out the results
- The result files are subsequently parsed into presentation markdown format and commend posted via Github REST api
- For the scoped task of posting a comment to the PR, `actions/github-script@v6` is preferred for simplicity
-  
