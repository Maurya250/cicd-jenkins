# CI/CD Pipeline with Jenkins & Docker on AWS EC2

This repository demonstrates a complete CI/CD pipeline using Jenkins, where an application is automatically built, pushed to Docker Hub, and deployed on an AWS EC2 instance using a Jenkinsfile.

---

## Project Overview

This project focuses on implementing a real-world CI/CD workflow using Jenkins Pipeline (Jenkinsfile).  
The goal is to automate the build, push, and deployment process instead of using manual or freestyle jobs.

---

## Tech Stack Used

- Jenkins  
- Docker & Docker Hub  
- AWS EC2 (Ubuntu)  
- Git & GitHub  
- Jenkinsfile (Pipeline as Code)

---

## Project Structure

.
├── Jenkinsfile  
├── Dockerfile  
├── package.json  
├── src/  
├── public/  
└── README.md  

---

## CI/CD Workflow

1. Code is pushed to GitHub  
2. Jenkins pulls the source code  
3. Docker image is built using Jenkins  
4. Image is pushed to Docker Hub  
5. Application is deployed as a Docker container on EC2  

---

## Prerequisites

Before starting, ensure you have:

- AWS EC2 instance (Ubuntu)
- Jenkins installed on EC2
- Docker installed on EC2
- Docker Hub account
- GitHub repository access

---

## Step-by-Step Deployment on AWS EC2 Using Jenkins

### Step 1: Launch EC2 Instance
- Launch an Ubuntu EC2 instance
- Open inbound ports:
  - 8080 for Jenkins
  - 80 or application port (as required)

---

### Step 2: Install Docker on EC2

sudo apt update  
sudo apt install docker.io -y  
sudo usermod -aG docker jenkins  
sudo usermod -aG docker ubuntu  
sudo systemctl restart docker  

---

### Step 3: Install Jenkins on EC2

sudo apt update  
sudo apt install openjdk-17-jdk -y  

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc  

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list  

sudo apt update  
sudo apt install jenkins -y  
sudo systemctl start jenkins  

Access Jenkins in browser:

http://<EC2-PUBLIC-IP>:8080

---

### Step 4: Add Docker Hub Credentials in Jenkins

- Go to Manage Jenkins → Credentials
- Add new credentials:
  - Kind: Username & Password
  - ID: docker-hub-creds
  - Username: Docker Hub username
  - Password: Docker Hub password

---

### Step 5: Create Jenkins Pipeline Job

- New Item → Pipeline
- Pipeline definition: Pipeline script from SCM
- SCM: Git
- Repository URL:

https://github.com/Maurya250/cicd-jenkins.git

- Script Path:

Jenkinsfile

---

### Step 6: Jenkinsfile Pipeline

The Jenkinsfile handles:
- Source code checkout
- Docker image build
- Push image to Docker Hub
- Deploy container on EC2

---

### Step 7: Run the Pipeline

- Click Build Now
- Pipeline stages will execute successfully
- Application will be deployed automatically

---

## Access the Application

http://<EC2-PUBLIC-IP>:<PORT>

Example:
http://13.xxx.xxx.xxx:80

---

## Key Learnings

- Why Jenkins Pipeline is preferred over freestyle jobs
- Pipeline as Code using Jenkinsfile
- Secure credential management in Jenkins
- Automated Docker build, push, and deployment
- Real-world CI/CD workflow on AWS EC2

---

## Author

Aniket Maurya  
DevSecOps | CI/CD | Jenkins | Docker | AWS
