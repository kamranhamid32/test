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
1. **Clone the repository**:
   ```bash
   git clone https://github.com/kamranhamid32/cicd-kube.git
   cd cicd-kube

