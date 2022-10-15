# Multi steps container Pipeline

- Run unitest.
- Run Build/compilation of application.
- Build image of binary.
- Push image to remote registry.
- Run instance of app in dind.
- Run Integration test.
- Run static code analysis on code.
- Run security scan on image (clair, anchor, stackrox).
- Dispose test instance.
- Deploy to lower environment. 
