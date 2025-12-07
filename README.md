# SimpleTimeService – Particle41 DevOps Challenge

This repository contains my submission for the Particle41 DevOps Team Challenge.  
The goal is to demonstrate DevOps engineering capabilities across:

- Minimal application development  
- Docker containerization with security best practices (non-root user)  
- AWS infrastructure provisioning using Terraform  
- ECS Fargate deployment with Application Load Balancer  
- CI/CD automation using Jenkins running on EC2  
- Proper documentation & reproducibility  

# Project Structure

├── app/ # Python microservice + Dockerfile
│ ├── main.py
│ ├── requirements.txt
│ └── Dockerfile
│
├── terraform/ # Terraform IaC for AWS infrastructure
│ ├── main.tf
│ ├── variables.tf
│ ├── outputs.tf
│ └── terraform.tfvars (user-provided)
│
└── Jenkinsfile # Automated CI/CD pipeline for build + deploy


#  SimpleTimeService – Application Overview

A minimal Python Flask microservice exposing two endpoints:

### GET `/`
Returns:
```json
{
  "timestamp": "2025-01-01T12:23:30Z",
  "ip": "CLIENT_IP"
}

### GET /health
Used by ALB for health checks:

{"status": "ok"}


The application runs inside a Docker container using a non-root user, following security best practices.

# Prerequisites

Before using or deploying this project, install the following tools:

✔ Docker
Install: https://docs.docker.com/engine/install/

✔ AWS CLI
Install: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

✔ Terraform
Install: https://developer.hashicorp.com/terraform/install

✔ Git
Install: https://git-scm.com/downloads

✔ AWS Account

Required services:

VPC, Subnets, IGW, NAT
ECS Fargate
IAM
Application Load Balancer
CloudWatch Logs
S3 (for Terraform remote state create bucket which you want to use for remote state and add in terraform block)

# Run Application Locally With Docker

-Build the Docker image:

cd app
docker build -t simple-time-service:latest .

-Run the container:
docker run -p 5000:5000 simple-time-service:latest

-Test:

curl http://localhost:5000/
curl http://localhost:5000/health

### Jenkins Setup on AWS EC2 – Complete Guide

Below are the steps to set up Jenkins on an EC2 instance and run fully automated CI/CD.

# Launch EC2 Instance for Jenkins

Recommended:

Amazon Linux 2 
Instance type: t3.medium

Attach IAM role with these permissions:

AmazonEC2FullAccess (optional)
AmazonECRFullAccess
AmazonS3FullAccess (if using remote backend)
CloudWatchFullAccess
IAMPassRole

Open inbound ports in ec2 security group:

8080 → Jenkins
22 → SSH

# Install Jenkins
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y

sudo wget -O /etc/yum.repos.d/jenkins.repo \
     https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins

Access Jenkins:
http://<EC2-Public-IP>:8080

Get initial password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Install Docker on EC2 (Required for CI/CD)

sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker jenkins
sudo usermod -aG docker ec2-user
sudo reboot

# Install AWS CLI on Jenkins EC2

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Install Terraform on EC2

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo \
    https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform

Jenkins CI/CD Pipeline (Automated Build + Deploy)

# This project contains a Jenkinsfile that performs:

Checkout repository
Detect AWS account ID
Build Docker image
Push image to Amazon ECR
Execute Terraform (init, plan, apply)
Output ALB URL

# Jenkins Job Setup

In Jenkins → New Item → Pipeline
Select Pipeline script from SCM
Enter repo URL:
https://github.com/pranjal124/SimpleTimeService.git
Branch: main
Script Path: Jenkinsfile

Run Build Now

Jenkins will:
✔ Build your microservice
✔ Push Docker image to ECR
✔ Deploy AWS infrastructure
✔ Start ECS service
✔ Output ALB DNS

===>access application using ALB URL

# Notes & Best Practices

Destroy resources when done to avoid AWS billing:
terraform destroy


Never commit AWS credentials to Git.
Use IAM roles for EC2 and Jenkins (recommended approach).
ECR repository creation is fully automated in pipeline.

