```bash
pipeline 
{ 
agent any 
```

**Agent and Tools**: The pipeline is configured to run on any available agent, and it defines the JDK and Maven versions to be used.
**Environment Variables**: It sets an environment variable SCANNER_HOME to the location of the SonarQube scanner tool.

```bash
    tools { 
        jdk 'jdk17' 
        maven 'maven3' 
    } 
    enviornment { 
        SCANNER_HOME= tool 'sonar-scanner' 
    } 
```
```bash
stages { 
```
Git Checkout: Check out the code from a Git repository.

```bash
    
        stage('Git Checkout') { 
            steps { 
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/Boardgame.git' 
            } 
        } 
```
**Compile**: Compile the project using Maven, Check if there is any Compilation error.
```bash      
        stage('Compile') { 
            steps { 
                sh "mvn compile" 
            } 
        } 
```   
**Test**: Run tests using Maven.
```bash
        stage('Test') { 
            steps { 
                sh "mvn test" 
            } 
        } 
```
File System Scan: Scan the whole source code so, that we can find vulneralibilities in dependencies (pom.xml), also id there is any kind of sensitive data stored inmy repository.         
```bash
        stage('File System Scan') { 
            steps { 
                sh "trivy fs --format table -o trivy-fs-report.html ." 
            } 
        } 
```
**SonarQube Analysis**: Perform static code analysis using SonarQube.  
    - SonarQube page -> Administration -> security -> users -> tokens -> name the token -> Generate -> copy
    - Jenkin -> dashboard -> manage jenkins -> credentials -> global -> add credentails -> select secret key -> paste token
    - Jenkin -> dashboard -> manage jenkins -> system -> sonarqube server -> Name: Sonar -> URL <SonarURL:9000> -> select token -> ID: sonar-cred -> Description: sonar-cred.

```bash
        stage('SonarQube Analsyis') { 
            steps { 
                withSonarQubeEnv('sonar') { 
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \ 
                            -Dsonar.java.binaries=. ''' 
                } 
            } 
        } 
```
**Quality Gate**: Wait for the quality gate status from SonarQube.  
    - Sonarqube -> administration -> configuration -> webhook -> create -> name: jenkins -> soarqube-webhook -> URL:<http//:jenkinsURL:8080>

```bash
        stage('Quality Gate') { 
            steps { 
                script { 
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'  
                } 
            } 
        } 
```
**Build**: Package the project using Maven.
```bash
        stage('Build') { 
            steps { 
               sh "mvn package" 
            } 
        } 
```
**Publish To Nexus**: Deploy artifacts to Nexus repository manager.  
Sonar-Nexus -> Browse -> copy maven releases and maven snapshot URL -> add the URL in pom.xml file
Dashboard -> managed files -> add a new config -> select global maven -> global setting
```bash
        stage('Publish To Nexus') { 
            steps { 
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: traceability: true) { 
                    sh "mvn deploy" 
                } 
            } 
        } 
```

**Build & Tag Docker Image**: Build and tag a Docker image.
```bash
        stage('Build & Tag Docker Image') { 
            steps { 
               script { 
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { 
                            sh "docker build -t aankusshh/boardshack:latest ." 
                    } 
               } 
            } 
        } 
```
**Docker Image Scan**: Scan the Docker image for vulnerabilities using Trivy.  
```bash
        stage('Docker Image Scan') { 
            steps { 
                sh "trivy image --format table -o trivy-image-report.html aankusshh/boardshack:latest " 
            } 
        } 
```
**Push Docker Image**: Push the Docker image to a Docker registry. 
```bash 
        stage('Push Docker Image') { 
            steps { 
               script { 
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') { 
                            sh "docker push aankusshh/boardshack:latest" 
                    } 
               } 
            } 
        }
```
**Deploy To Kubernetes**: Deploy the application to Kubernetes. 
```bash
        stage('Deploy To Kubernetes') { 
            steps { 
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') { 
                    sh "kubectl apply -f deployment-service.yaml" 
                } 
            } 
        } 
```
**Verify the Deployment**: Check the status of pods and services in the Kubernetes cluster.  
```bash
        stage('Verify the Deployment') { 
            steps { 
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') { 
                    sh "kubectl get pods -n webapps" 
                    sh "kubectl get svc -n webapps" 
                } 
            } 
        } 
  
         
    } 
```
**Post-Build Actions**: After each run, an email is sent with the build status and a link to the Jenkins console output. Additionally, it attaches the Trivy image report.
RBAC (Role based access control): Role-Based Access Control (RBAC) assigns permissions to roles rather than individual users. Users are then assigned roles based on their job functions or responsibilities. Access control policies are enforced based on these role assignments. RBAC simplifies access management, enhances security, and ensures compliance with regulatory requirements. It provides granular control over system access while reducing the risk of unauthorized access.

```bash
    post { 
        always { 
            script { 
                def jobName = env.JOB_NAME 
                def buildNumber = env.BUILD_NUMBER 
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN' 
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red' 
 
                def body = """ 
                    <html> 
                    <body> 
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;"> 
                    <h2>${jobName} - Build ${buildNumber}</h2> 
                    <div style="background-color: ${bannerColor}; padding: 10px;"> 
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3> 
                    </div> 
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p> 
                    </div> 
                    </body> 
                    </html> 
                """ 
 
                emailext ( 
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}", 
                    body: body, 
                    to: 'ankush56singh@gmail.com', 
                    from: 'jenkins@example.com', 
                    replyTo: 'jenkins@example.com', 
                    mimeType: 'text/html', 
                    attachmentsPattern: 'trivy-image-report.html' 
                ) 
            } 
        } 
    } 
} 
```