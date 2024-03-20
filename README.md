# Steps Followed in the Project.

We will create this project in four phases:

**Phase 1** (Make three EC2 instances)
- Jenkins (install jenkins) 
- SonarQube
- Nexus Repository
- WebApp (Make 3 Ec2 instance, install kubernetes (1 master and 2 slave) + Docker)

**Phase 2** (Making and Configuring Git)

**Phase 3** (Build Pipelines in Jenkins)

**Phase 4** (Setting up Monitoring Tools)


## Phase 1 : Setup instances and install software
- Make 6 instances [Jenkins, SonarQube, Nexus Repository, Kubernetes ( 1 Master, 2 Slave )]
- Secure them with a proper security group and VPC (GOOD PRACTICE)
- Now set up Kubernetes server by installing Docker, Kubernetes, KubeCTL, Kubelet and Kubeamd (Shell-Script for installing all these are present in my DevOps 2024 repository )
- We will also install KubeAudit for checking security issues in Kubernetes.
  **Now we will setup SonarQube and Nexus server**
  - Install Docker in both servers (Shell-Script for installing present in my DevOps 2024 repository)
  - Check if docker got install by **docker pull Hello-World**
  - Now Create a **Container** in both Servers
  - Provide port 9000:9000 to sonarQube with image **sonarqube: lts-community** (docker run -d --name container_name -p 9000:9000 sonarqube: lts-community)
  - Provide port 8081:8081 to nexus with image **sonatype/nexus3** (docker run -d --name container_name -p 8081:8081 sonatype/nexus3)
  - Now for confirmation that the docker container have been made type **docker ps**
- Now we move to Jenkins server
  - Java should be present in compute to install jenkins, So first install java
  - Install Docker (Shell-Script for installing present in my DevOps 2024 repository)

**NOW WE ARE SET WITH ALL THE SERVERS AND READY TO MAOVE TO PHASE TWO**


## Phase 2 : Setup a Git Repository
- Create a new repository
- Make it private (you can make it public once the project is done but as we are making a corporate level project so make it private)
- Follow these Steps in your system [git clone -> make files and directories -> git add . -> git status -> git commit -> git push]


## Phase 4 (Build Pipelines in Jenkins)
- **Install Plugin** (Dashboard -> Manage jenkins -> Plugins)
  - JDK (Eclipse Temurin Installer) -> Used when we want different versions of JDK to support.
  - Maven (config file provider)
  - maven (pipeline maven integration)
  - maven (maven integration)
  - sonar (sonarqube scanner) -> perform the analysis
  - sonar (sonarqube server) -> where the result will be shown
  - docker (docker)
  - docker (docker pipeline)
  - docker (docker-build-step)
  - kubernetes (kubernetes CLI)
  - kubernetes (kubernetes client API)
  - kubernetes (kubernetes credentials)
- **Now we have to configure the tools** (Dashboard -> manage jenkins -> Tools)
  - JDK installer
    - Name -> jdk17
    - install automatically -> adoptium.net -> choose any version
  - SonarQube Scanner installer
    - Name -> sonar-scanner
    - version -> latest
  - Maven installer
    - Name -> maven3
    - Version -> choose
  - Docker installer
    - Name -> docker
    - Version -> Latest
 - **Construct Pipeline** (Dashboard -> New Item -> select Pipeline)
   - Discard old builds (set it to the 3) -> No of previous build it will show.
   - I will recommend you to use pipeline syntax to construct the pipeline (But if you are finding any difficulty in constructing the pipeline you can see the file names pipeline.md in the repository).

  ``` hiii ```
