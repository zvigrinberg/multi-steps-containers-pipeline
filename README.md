## Multi steps container CI/CD Pipeline

Note : Triggering of the docker/podman build will be performed by git post-commit hook script:

1. First MultiStages Build(Dockerfile/Container file multistage build):
- Run unitest.
- Run Build/compilation of application.
- Run static code analysis on code.
- Build image of binary.

- Push image to remote registry (phase from script).

2. Second MultiStage Build:

- Run security scan on image (clair, anchore, stackrox).
- Run instance of app in dind.
- Run Integration test With curl/wget tools
- Dispose test instance in dind

3. Third Phase from script:
- Deploy to lower environment.
- Implement advanced deployment models like canary deployment and blue/green deployment
  Using nginx to achieve zero Downtime(and hence possible at peak application usage to perform) and gradually pass all traffic to new version instance, and to enable immediate rollback capabilities in case something went wrong



## Pre-requisites

1. Spinning up a sonatype nexus server instance using Podman/docker.
```shell
# Makes ports 80 and 443 boundables
echo net.ipv4.ip_unprivileged_port_start=80 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
# Run Nexus instance with volume mounted to local host filesystem.
podman run --cap-add CAP_NET_BIND_SERVICE -d -p 8081:8081 -p 80:80 -p 5000:5000 -p 443:443 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
# Once server is up, fetch admin password 
 export NEXUS_ADMIN_PASSWORD=podman exec nexus cat /nexus-data/admin.password
```

2. Create DNS Record for 127.0.0.1:
```shell
echo "127.0.0.1 127.0.0.1.podman.io" | sudo tee -a /etc/hosts
# If /etc/hosts already contains an entry for 127.0.0.1 , just add to its line at the end - " , 127.0.0.1.podman.io "
```
3. open sonatype nexus Web UI
```shell
xdg-open http://127.0.0.1.podman.io:8081
```
 
 4. Sign in to nexus using credentials admin/$NEXUS_ADMIN_PASSWORD
 
 5. Go to machine wheel at top --> on the left panel click on Repositories --> Click on +Create Repository --> Choose Docker(Hosted), and then populate all of the following, at the end, click on Button `Create repository`:
    - [x] HTTP
    - [ ] HTTPS
    - [x] Allow anonymous docker pull ( Docker Bearer Token Realm required ) 
 ```properties
 Name=podman-hub
 HTTP=80
 ```
 6. Go to Realms on the left panel of Nexus --> Click on Docker Bearer Token Realm to move it from available to Active.
 
 7. Connect to nexus registry with podman
 ```shell
 podman login --tls-verify=false 127.0.0.1.podman.io -u admin -p $NEXUS_ADMIN_PASSWORD
 ```
 
 8. pull from docker hub some image and push it to the new registry
 ```shell
 podman pull nginx
 podman tag docker.io/library/nginx:latest 127.0.0.1.podman.io/podman-hub/nginx:latest
 podman push --tls-verify=false 127.0.0.1.podman.io/podman-hub/nginx:latest
 ```
 
