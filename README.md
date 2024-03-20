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
