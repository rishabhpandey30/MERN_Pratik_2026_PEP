CI/CD Pipeline: Jenkins, SonarQube & Nexus, Docker & AWS
--------------------------------------------------------------
1.Create EC2 Instance (AWS)

Go to Amazon Web Services Console
Launch Instance:
AMI: Ubuntu (22.04 recommended)
Running Jenkins, SonarQube, and Nexus simultaneously requires more resources.
Instance Type: Upgrade to t3.medium (minimum 4GB RAM).
A t2.micro will crash immediately with Nexus.
Key pair: create/download .pem
Security Group:
SSH → Port 22
Custom TCP → Port 8080 (for Jenkins)
Custom TCP (9000): SonarQube UI
Custom TCP → Port 8181 (or 8081): For Nexus Repository Manager.
Custom TCP → Port 3000 (or your app port): This is mandatory to see your live website in the browser.
----------------------------------
2.Connect to EC2 (SSH)

ssh -i your-key.pem ubuntu@ your public ipv4 address
-----------------------------------
3.Install Docker
sudo apt update
sudo apt install docker.io -y

(a).Start and enable Docker:

sudo systemctl start docker
sudo systemctl enable docker

(b). Add Docker Permissions:
  For group activation:-
  sudo usermod -aG docker ubuntu
  newgrp docker

Instead of exiting, newgrp docker applies permissions instantly.

(c).Verify Docker

docker --version
docker run hello-world
---------------------------------------------------
4. (a)Run Jenkins Container

docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  --restart always \
  jenkins/jenkins:lts

  (b)Run SonarQube for Code Quality checks

docker run -d --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  --restart always \
  sonarqube:lts-community

  (c)Run Nexus Container

docker run -d --name nexus \
  -p 8081:8081 \
  --restart always \
  sonatype/nexus3

Note: Nexus is resource-heavy. Ensure your instance has at least 4GB of RAM.
      It takes 2-3 minutes to start up. Check logs with: docker logs -f nexus

Q.How to verify they are alive ?
Run the following command to see if they are up:docker ps

After it open the public ip and add jenkins port to set admin passwaord.
-----------------------------------------
5.Get Jenkins Admin Password

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
-----------------------------------------
6.Access Jenkins in Browser

http://<your-ec2-public-ip>:8080
Paste the password → Continue setup
-----------------------------------------
7.Install Plugins
Click Install Suggested Plugins
Create admin user

Required Extra Plugins:-
Docker Pipeline
SonarQube Scanner
NodeJS(if using JavaScript-based projects)
Nexus Artifact Uploader

Final Permission Check (PowerShell):
Before you run your first build, run this command in your SSH session to prevent the "Permission Denied"
sudo chmod 666 /var/run/docker.sock
---------------------------------------------
8.(a)SonarQube Integration:
Access http://98.94.102.71:9000 (Login: admin / admin).  
Go to My Account > Security and generate a Global Analysis Token.

The Handshake (Jenkins Configuration):

Add Credentials: Go to Manage Jenkins > Credentials > System > Global credentials.
Kind: Select Secret text.
Secret: Paste your SonarQube Token.
ID: Name it sonar-token (you will use this ID in your pipeline).
Configure System: Go to Manage Jenkins > System.
Find the SonarQube installations section.
Name: Enter sonar-server.
Server URL: Enter http://<EC2-IP>:9000.
Server authentication token: Select the sonar-token you just created from the dropdown.
Install Scanner: Go to Manage Jenkins > Tools.
Find SonarQube Scanner.
Click Add SonarQube Scanner.
Name: Enter sonar-scanner.
Check Install automatically.

(b)Nexus Initial Configuration
Access: http://98.94.102.71:8081
Password: Click "Sign In." The default user is admin. To get the password:-
docker exec nexus cat /nexus-data/admin.password
Setup Wizard: Follow the prompts to set a new password and enable Anonymous Access (for simplicity in this demo).

Create a Raw Hosted Repository:
Navigation: Go to Server Administration (cog icon) > Repositories > Create repository.
Recipe Selection: Select raw (hosted). (This is best for static files and .tar archives).
Name: Enter my-frontend-repo.
Deployment Policy: Ensure it is set to Allow redeploy if you plan to push updated versions of the same build.

(c).Integrate Nexus Credentials in Jenkins
To allow Jenkins to upload artifacts to Nexus, you must store the Nexus login details securely.
1. Navigate to Credentials:
From the Jenkins Dashboard, go to Manage Jenkins > Credentials.
Click on the (global) domain link.

2. Add the Secret:
Click Add Credentials on the left sidebar.
Kind: Select Username with password.
Username: admin.
Password: (The new password you set during Nexus setup).
ID: Enter nexus-creds. CRITICAL: This ID must exactly match the credentialsId used in your Jenkinsfile.
Description: Nexus Repository Credentials. 
----------------------------------------------
9.The Jenkins Pipeline (Jenkinsfile)
Create a Jenkinsfile in your repository root to automate the build:

```
pipeline {
    agent any

    environment {
        // Nexus Configuration
        NEXUS_VERSION = 'nexus3'
        NEXUS_PROTOCOL = 'http'
        NEXUS_URL = '54.173.95.16:8081'
        NEXUS_REPOSITORY = 'my-frontend-repo'
        NEXUS_CREDENTIAL_ID = 'nexus-creds'
        
        // Docker Configuration
        IMAGE_NAME = "my-frontend-app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kPratik07/devops-frontend.git'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                    sh "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Package for Nexus') {
            steps {
                // Fixed syntax: exclude comes first to avoid self-reference error
                sh 'tar --exclude=".git" --exclude="frontend-build.tar" -cvf frontend-build.tar .'
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: 'com.frontend',
                        version: '1.0.' + env.BUILD_NUMBER,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: 'frontend-app', classifier: '', file: 'frontend-build.tar', type: 'tar']
                        ]
                    )
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```
---------------------------------------------------------
11.Define the Quality Gate in SonarQube:

SonarQube is running, but it needs to tell Jenkins to "stop" if the code is bad.
In SonarQube: Go to Quality Gates and ensure the "Sonar way" is set as default.
In Jenkinsfile: You can add a specific step to wait for the analysis result:
```
stage("Quality Gate") {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```
This ensures that if the code has security vulnerabilities, the "Build & Deploy" stage never runs.
--------------------------------------------------------------------
12.How to Verify Your App in Nexus
Once the Jenkins pipeline shows a green "Success" for the Upload to Nexus stage, follow these steps to find your application:
Access the UI: Open your browser and go to http://<EC2-IP>:8081.
Navigate to Browse: Click on the Browse (cube icon) in the left-hand sidebar.
Select Repository: From the list of repositories, click on the one you created, e.g., my-frontend-repo.
Drill Down the Path: You will see a folder structure based on the groupId and version defined in your Jenkinsfile:
Click on com.
Click on frontend.
Click on frontend-app.
Find the Artifact: Open the folder matching your Jenkins build number (e.g., 1.0.7).
Identify the Files: You should see your frontend-app-1.0.7.tar file along with its .md5 and .sha1 checksum files.

You can finally see your application running on :http://<YOUR-EC2-PUBLIC-IP>:8085