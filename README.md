# AWS CI/CD Pipeline | Deploy Production-Grade Full-Stack App | CodeBuild, CodeDeploy, CodePipeline

This project demonstrates a **production-grade 3-tier architecture** for deploying a full-stack React + Node.js + MySQL application on AWS using **CI/CD pipelines** with CodeBuild, CodeDeploy, and CodePipeline.

---

## Architecture Overview

1. Create your own VPC
2. Create Security Groups
3. Create S3 Bucket for code/artifacts
4. Create IAM Roles
5. Create RDS MySQL
6. Create App Server and setup application
7. Create Internal Load Balancer for App Servers
8. Create Web Server and setup Nginx
9. Create External Load Balancer for Web Servers
10. Setup Auto Scaling Groups (ASG) for Web and App Servers
11. Store secrets in Parameter Store
12. Create S3 Private Bucket for artifacts
13. Setup HTTPS and Route53

---

## Build & Deployment Files

- **`buildspec.yml`**: Defines the build process for AWS CodeBuild including dependencies, tests, build artifacts, and deployment instructions.
- **`appsec.yml`**: Used in CodePipeline for application-related operations like install, move, copy, etc.

---

## Step-by-Step Implementation

### 1. Create a VPC

- Name: `3-tier-project`
- CIDR: `192.168.0.0/16`
- Enable DNS hostnames
- Create IGW: `3-tier-project-IGW` and attach to VPC
- Subnets:
  - **Public:** 192.168.1.0/24, 192.168.2.0/24
  - **Private:** Web 192.168.3.0/24, 192.168.4.0/24; App 192.168.5.0/24, 192.168.6.0/24; DB 192.168.7.0/24, 192.168.8.0/24
- NAT Gateway: `3-tier-project-NAT` in PublicSubnet-1a
- Route Tables: Public and Private routing
- Security Groups: Bastion, Web ALB, Web Server, App ALB, App Server, Database

---

### 2. Launch Bastion Host

- AMI: Amazon Linux 2023
- Public Subnet, Bastion-SG

---

### 3. IAM Roles

- **EC2:** `3-tier-ec2-role` (CloudWatch, CodeDeploy)
- **CodeBuild:** `3-tier-codebuild-role` (S3, CloudWatch, SSM)
- **CodeDeploy:** `3-tier-codedeploy-role`
- **CodePipeline:** `3-tier-codepipeline-role`

---

### 4. RDS MySQL Setup

- DB Identifier: `dev-db-instance`
- VPC: `3-tier-project`
- Subnet Group: `tier-Subnet-Group`
- SG: `Database-SG`
- Username: `admin`
- Password: `root123456`

---

### 5. Setup Database from Bastion

- Install MySQL Client
- Connect to RDS and create databases
- Setup schema from `backend/db` folder

---

### 6. Application Tier Setup

- Create Launch Template: `application-tier-LT`
- Empty Target Group: `app-tier-tg` (port 3200)
- Internal ALB: `app-tier-internal-alb`
- Auto Scaling Group: `application-tier-ASG`

---

### 7. Web Tier Setup

- Launch Template: `web-tier-LT`
- Empty Target Group: `web-tier-tg` (port 80)
- External ALB: `web-tier-external-alb`
- Auto Scaling Group: `web-tier-ASG`

---

### 8. Parameter Store

- Store DB credentials and endpoint:
  - `/nodeapp/db/hostname`
  - `/nodeapp/db/name`
  - `/nodeapp/db/password`
  - `/nodeapp/db/port`
  - `/nodeapp/db/user`

---

### 9. S3 Private Bucket

- Bucket Name: `3-tier-project-codebuild-artifacts`

---

### 10. CI/CD Setup: Application Tier

- **CodeBuild:** `backend-build` (uses `backend/buildspec.yml`)
- **CodeDeploy:** `backend-deploy` with Deployment Group `backend-Deployment-grp`
- **CodePipeline:** `Backend-Pipeline`

---

### 11. CI/CD Setup: Web/Presentation Tier

- **CodeBuild:** `frontend-build` (uses `frontend/buildspec.yml`)
- **CodeDeploy:** `frontend-deploy` with Deployment Group `frontend-Deployment-grp`
- **CodePipeline:** `frontend-Pipeline`

---

### 12. CloudFront & Route53

- Setup HTTPS and DNS routing for your application

---

## Useful Links

- [AWS CodeBuild Webhooks](https://codebuild.ap-south-1.amazonaws.com/webhook)

---

## Repository

[aws-cicd-react-node-mysql-app](https://github.com/Shaikarif143/aws-cicd-react-node-mysql-app)

---

## Notes

- Ensure all EC2 instances, ALBs, and ASGs are configured with correct subnets and security groups.
- Use Parameter Store for sensitive information instead of hardcoding credentials.
- Monitor logs in CloudWatch for both application and infrastructure components.
