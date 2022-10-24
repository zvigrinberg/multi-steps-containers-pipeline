8# Multi steps container CI/CD Pipeline

- Run unitest.
- Run Build/compilation of application.
- Run static code analysis on code.
- Build image of binary.
- Push image to remote registry.
- Run instance of app in dind.
- Run Integration test.
- Run security scan on image (clair, anchore, stackrox).
- Dispose test instance.
- Deploy to lower environment.
- Implement advanced deployment models like canary deployment and blue/green deployment
