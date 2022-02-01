## Demo Repo
### The repo contains the following elements
1. At root level spring-boot `perso-greet` project ( [tutorial](https://spring.io/guides/gs/perso-greet/) ) 
   * Maven project - build using `mvn clean package`
   * Dockerfile - Packaging this project in Image  
   * Jenkinsfile - CI/CD for this project  
2. helm-package
   * Contains greet chart - basic web helm package to wrap the docker image container for k8s deployment
   * Dockerfile_helm - docker image for building the helm package  
3. deployment folder 
   * containing ansible(v2.9) playbooks and roles 
     - Setup the Jesnkins Master server
     - Setup the k8s (minikube) deployment server
     - Deploying the app via helm to k8s (localhost minikube)
     - Starting + Stopping port-forward from remote to the minikube service 
4. environment folder 
   * Containing the info to access both servers 
    
### Status
1. perso-greet project (maven) is building successfully.
2. The perso-greet Project Image is also builds correctly and tested locally. 
3. Jenkinsfile was tested in a previous environment which has been compromised
4. deployment 
   1. deploy_jenkins playbook deploying jenkins master 
   2. deploy_minikube deploying the runtime environment
   3. deploy_app_to_minikube.yml deploy the app to minikube

### Currently, the original Jenkins master is not safe to run the build. Thus, Jenkinsfile is our only source of truth for the required configuration.   
### The repo in this zip file is our only leftover we could find, it was also compromised and many untested changes were committed 
### Your mission, should  you choose to accept it
1. Deploy a new Jenkins master using deployment playbook 
   - Jenkins configuration uses [configuration-as-code-plugin](on-as-code-pluhttps://github.com/jenkinsci/configuratigin/blob/master/README.md) to start secured
2. Deploy the minikube development environment via playbook
3. Run the Jenkinsfile and make it successfully finish
    - Connect the Jenkins to a git VCS (i.e. GitHub/bitbucket/gitlab 
      all has free account capacity) 
    - Use webhook for each commit push (no polling) 
    - Use jenkins credentials when needed (someone always hacking)
4. Run the `deploy_app_to_minikube.yml` playbook development to deploy the `greet` app with helm to minikube development env
5. Remote stage test - ***Optional***
   - Enable remote test for the greet api from Jenkinsfile stage
6. SMTP - ***Optional***
   - Enable smtp mailing for post stage

##### Resources:
***In the environment folder***
1. jkey - the private key for servers connect with default ec2 user
2. ips - list of ips to use with the key one for Jenkins and one for deployment

### Delivery
#### Document all your steps for the solution so we can reuse them
#### Send us back a zip with your changes and a readme with all prerequisites and action/scripts needed to redeploy it on a clean env

### The environment will self-destruct at ...

###### Note on jenkins deployment
> Since there are problems with running docker in docker ([you can read about it here](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/))
> Please use the following host mounts in your docker configuration and not docker in docker
> as it's simpler than jenkins.io installation guide
>
```yaml
  - /var/run/docker.sock:/var/run/docker.sock
  - /usr/local/bin/docker:/usr/local/bin/docker
```
> Make sure the runtime user and group has docker socket permissions !! 
