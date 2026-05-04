# demo-devops-simulation
This repo is for end-to-end devops project

## Overview
This project demonstrates a complete DevOps workflow implementing CI/CD pipeline, containerization, and Kubernetes-based deployment on AWS.
The pipeline automates application build, Docker image creation, and deployment to Kubernetes

## 🧠 Architecture
GitHub → Jenkins → Docker → AWS ECR → Kubernetes (EKS) → Application


## Setup Instructions

## Install Dependencies

Run the following commands on your system:


# Install

sudo apt update
sudo apt install docker.io git -y

# Install Kubectl

url -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Terraform
sudo apt install unzip -y
wget https://releases.hashicorp.com/terraform/1.0.0/terraform_1.0.0_linux_amd64.zip
unzip terraform_1.0.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Create Sample Application

mkdir devops-project
cd devops-projec
nano app.py
# Flask
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "DevOps Project Running 🚀"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

# setup docker
#create dockefile
nano Dockerfile

FROM python:3.8
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]

# Build and Run locally
docker build -t devops-app .
docker run -d -p 5000:5000 devops-app

# test
http://localhost:5000

# Push Image to AWS ECR
aws ecr create-repository --repository-name devops-app
#Login
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-south-1.amazonaws.com

#Tag and Push
docker tag devops-app:latest <account-id>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:latest
docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:latest

# Kubernetes deployment
mkdir k8s
cd k8s

# Create deployment.yaml file 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-app
  template:
    metadata:
      labels:
        app: devops-app
    spec:
      containers:
      - name: devops-app
        image: <ecr-image-url>
        ports:
        - containerPort: 5000
# Create service.yaml file
apiVersion: v1
kind: Service
metadata:
  name: devops-service
spec:
  type: NodePort
  selector:
    app: devops-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30007

# Apply
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
# Jenkins CI/CD pipeline
# Install Jenkins
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y

# Jenkinsfile
#create
nano Jenkinsfile
#paste below code
pipeline {
    agent any

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/your-repo/devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-app .'
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker tag devops-app:latest <ECR-URL>
                docker push <ECR-URL>
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}

# Monitoring
kubectl create namespace monitoring

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring


# ######################## DevOps End-to-End Project flow ###############################
## Tech Stack
- AWS (EKS, ECR)
- Jenkins
- Docker
- Kubernetes
- Terraform (optional)

## Features
- Automated CI/CD pipeline
- Docker containerization
- Kubernetes deployment
- Monitoring setup

## Architecture
GitHub → Jenkins → Docker → ECR → EKS

## Output
Application deployed on Kubernetes cluster with zero downtime deployment

# Explaination about project
Built an end-to-end CI/CD pipeline using Jenkins, Docker, AWS ECR, and Kubernetes (EKS), automating deployment from code commit to production
Deployed containerized applications on Kubernetes with zero-downtime deployment strategies
Implemented Infrastructure as Code and monitoring setup for application reliability

