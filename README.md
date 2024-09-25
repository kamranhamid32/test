# CI/CD Pipeline for Kubernetes

![image](https://github.com/user-attachments/assets/2916b298-e631-4a43-8310-4e86ae5f7237)



In this project, the process begins when developers push new changes to a GitHub repository containing the source code. Once the changes are committed to the repository, my role as a DevOps engineer begins with the creation of a CI/CD pipeline using Jenkins.

## Pipeline Overview:
Compilation Stage:
The first stage in the Jenkins pipeline is the compilation of the source code. This step is essential to catch any syntax-based errors early on in the process.

## Unit Testing Stage:
Once the compilation is successful, the next step is to run unit tests. These tests ensure that the functionality of the code meets the required specifications and behaves as expected.

## SonarQube Code Quality Check:
In this stage, we integrate SonarQube to perform a code quality check. The goal here is to identify any bugs, code smells, vulnerabilities, or issues in the source code. A cleaner codebase with fewer issues translates to better overall quality.

## Vulnerability Scanning:
After checking the code quality, we run a vulnerability scan using Trivy. This tool scans both the application source code and its dependencies to detect sensitive data leaks or outdated, vulnerable dependencies. Instead of generating reports directly in the console (which is not ideal for analysis), we export these reports in formats like HTML or text for easier inspection and review.

## Building and Packaging the Application:
Once the code passes all checks, we proceed to build the application and package it to create the application artifact. This artifact is then published to a Nexus repository to facilitate version control and release management.

## Docker Image Creation and Tagging:
After publishing the artifact, we create a Docker image of the application and tag it appropriately to manage different versions. The tagged image is scanned again using Trivy to check for container vulnerabilities before pushing it to a Docker Hub repository, which could be either private or public depending on the project’s requirements.

## Kubernetes Cluster Security Audit:
Before deploying the application to Kubernetes, we ensure that the Kubernetes cluster is secure. We use a tool called Kube-Audit to scan the cluster and identify any potential security issues.

## Application Deployment:
The application is then deployed to the Kubernetes cluster. We verify the success of the deployment and ensure that the application is running as expected.

## Notification and Monitoring:
The final stage of the pipeline includes email notifications to inform whether the pipeline succeeded or failed. Once the application is deployed, we monitor its performance using Node Exporter.



## Setup
### 1. **Jenkins Setup** on Ubuntu 
      
      #!/bin/bash
      sudo apt update
      
      sudo apt install openjdk-11-jdk -y
      sudo apt install openjdk-8-jdk -y
      sudo apt install maven -y

      sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
        https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
      
      echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
        https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
        /etc/apt/sources.list.d/jenkins.list > /dev/null

      sudo apt-get update
      
      sudo apt-get install jenkins -y
      
      # Add Docker's official GPG key:
      
      sudo apt-get update
      sudo apt-get install ca-certificates curl
      
      sudo install -m 0755 -d /etc/apt/keyrings
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
      sudo chmod a+r /etc/apt/keyrings/docker.asc


      # Add the repository to Apt sources:
      
      echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      
      sudo apt-get update
      
      sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

## 2. **Nexus Installation**  on CentOS9
      #!/bin/bash

      sudo rpm --import https://yum.corretto.aws/corretto.key
      sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
      
      sudo yum install -y java-17-amazon-corretto-devel wget -y
      
      mkdir -p /opt/nexus/   
      mkdir -p /tmp/nexus/                           
      cd /tmp/nexus/
      NEXUSURL="https://download.sonatype.com/nexus/3/latest-unix.tar.gz"
      wget $NEXUSURL -O nexus.tar.gz
      sleep 10
      EXTOUT=`tar xzvf nexus.tar.gz`
      NEXUSDIR=`echo $EXTOUT | cut -d '/' -f1`
      sleep 5
      rm -rf /tmp/nexus/nexus.tar.gz
      cp -r /tmp/nexus/* /opt/nexus/
      sleep 5
      useradd nexus
      chown -R nexus.nexus /opt/nexus 
      cat <<EOT>> /etc/systemd/system/nexus.service
      [Unit]                                                                          
      Description=nexus service                                                       
      After=network.target                                                            
                                                                        
      [Service]                                                                       
      Type=forking                                                                    
      LimitNOFILE=65536                                                               
      ExecStart=/opt/nexus/$NEXUSDIR/bin/nexus start                                  
      ExecStop=/opt/nexus/$NEXUSDIR/bin/nexus stop                                    
      User=nexus                                                                      
      Restart=on-abort                                                                
                                                                        
      [Install]                                                                       
      WantedBy=multi-user.target                                                      
      
      EOT
      
      echo 'run_as_user="nexus"' > /opt/nexus/$NEXUSDIR/bin/nexus.rc
      systemctl daemon-reload
      systemctl start nexus
      systemctl enable nexus
      
1. **Clone the repository**:
   ```bash
   git clone https://github.com/kamranhamid32/cicd-kube.git
   cd cicd-kube


