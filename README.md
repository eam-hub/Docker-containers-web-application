# Web Application Deployment on AWS with Docker, ECS, and Fargate

## Project Overview

This project demonstrates the deployment of a containerized web application on AWS using Docker and Amazon ECS with Fargate. The infrastructure was designed using a highly available three-tier architecture spanning multiple Availability Zones.

The application Docker image is stored in Amazon ECR and deployed to ECS Fargate tasks behind an Application Load Balancer. Supporting infrastructure includes a custom VPC, public and private subnets, NAT Gateway, Route 53, ACM SSL certificates, RDS MySQL, AWS Secrets Manager, and EC2 Instance Connect Endpoint (EICE) for secure administration.


![image](https://github.com/user-attachments/assets/2b9ab5c8-9270-4d8f-9af6-db049cf41432)

---

## Architecture

### AWS Services Used

* Amazon VPC
* Public and Private Subnets
* Internet Gateway
* NAT Gateway
* Security Groups
* EC2 Instance Connect Endpoint (EICE)
* Amazon S3
* AWS IAM
* Amazon Route 53
* AWS Certificate Manager (ACM)
* Amazon RDS MySQL
* AWS Secrets Manager
* Docker
* Amazon Elastic Container Registry (ECR)
* Amazon Elastic Container Service (ECS)
* AWS Fargate
* Application Load Balancer (ALB)
* Amazon CloudWatch

---

## Architecture Flow

```text
Users
   |
Route53
   |
Application Load Balancer
   |
ECS Fargate Tasks
   |
Amazon RDS MySQL

Container Images
   |
Amazon ECR

Application Code
   |
Amazon S3

Secrets
   |
AWS Secrets Manager
```

---

# Prerequisites

Before starting, ensure the following tools and accounts are available:

* AWS Account
* GitHub Account
* Git
* Visual Studio Code
* Docker Desktop
* Docker Hub Account
* AWS CLI
* Flyway Database Migration Tool
* SSH Key Pair
* Private GitHub Repository

---

# VPC and Networking Setup

## Create VPC

Create a VPC and six subnets across two Availability Zones:

### Public Subnets

* Public Subnet AZ1
* Public Subnet AZ2

### Private Application Subnets

* Private App Subnet AZ1
* Private App Subnet AZ2

### Private Database Subnets

* Private DB Subnet AZ1
* Private DB Subnet AZ2

Enable:

* DNS Hostnames
* DNS Resolution

---

## Internet Gateway

Create and attach an Internet Gateway to the VPC.

Create a route table for public subnets:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

---

## NAT Gateway

Create:

* Elastic IP
* NAT Gateway in Public Subnet AZ1

Create a route table for private application subnets:

```text
Destination: 0.0.0.0/0
Target: NAT Gateway
```

---

# Security Groups

## EC2 Instance Connect Endpoint

Inbound:

* Default

Outbound:

* SSH (22) to VPC CIDR

---

## Application Load Balancer

Inbound:

* HTTP (80) from 0.0.0.0/0
* HTTPS (443) from 0.0.0.0/0

Outbound:

* Default

---

## Web Server / ECS Tasks

Inbound:

* HTTP (80) from ALB Security Group
* HTTPS (443) from ALB Security Group

Outbound:

* Default

---

## Database Migration Server

Inbound:

* SSH (22) from EICE Security Group

Outbound:

* Default

---

## RDS Database

Inbound:

* MySQL/Aurora (3306)
* Source:

  * Web Server Security Group
  * Database Migration Server Security Group

Outbound:

* Default

---

# EC2 Instance Connect Endpoint (EICE)

Instead of using a Bastion Host, create an EC2 Instance Connect Endpoint inside the private application subnet.

Benefits:

* No public jump server required
* Secure access to private resources
* Reduced attack surface

---

# Amazon S3

Create an S3 bucket and upload the application package.

The application will retrieve deployment assets from S3 during provisioning.

---

# IAM Configuration

## Create S3 Access Policy

Permissions:

* Read application files from S3

## Create Secrets Manager Policy

Permissions:

* Retrieve application secrets

## Create EC2 IAM Role

Attach:

* S3 Policy
* Secrets Manager Policy

Assign the role to the Database Migration EC2 instance.

---

# Domain Registration and SSL

## Route 53

Register a domain name.

## AWS Certificate Manager

Request an SSL certificate for the domain.

Validate the certificate through Route 53.

---

# Database Layer

## Create DB Subnet Group

Use the private database subnets.

## Create RDS MySQL Instance

Configure:

* Database Name
* Master Username
* Master Password

Store credentials securely.

---

# AWS Secrets Manager

Store:

* Database Username
* Database Password
* GitHub Personal Access Token

Application containers retrieve secrets during runtime.

---

# Database Migration Server

Launch a temporary EC2 instance:

* Private Subnet
* IAM Role Attached
* Database Migration Security Group Attached

Install and execute Flyway migrations through User Data scripts.

---

# IAM User for CLI Access

Create an IAM User:

```text
AWS Console
→ IAM
→ Users
→ Create User
```

Attach:

```text
AdministratorAccess
```

Create Access Keys and configure AWS CLI:

```bash
aws configure --profile username
```

---

# Generate SSH Keys

Generate an SSH key pair:

```bash
ssh-keygen -t rsa -b 2048
```

Example location:

```text
C:\Users\username\.ssh\id_rsa
```

Add the public key to GitHub:

```text
GitHub
→ Settings
→ SSH and GPG Keys
→ New SSH Key
```

Clone the private repository:

```bash
git clone <repository-url>
```

Create a GitHub Personal Access Token and store it in AWS Secrets Manager.

---

# Docker Setup

Create a separate repository for Docker configuration.

Clone the repository:

```bash
git clone <repository-url>
cd <repository>
code .
```

Create a Dockerfile for the application.

---

## Build Docker Image

```bash
docker build \
--build-arg PERSONAL_ACCESS_TOKEN=<token> \
--build-arg GITHUB_USERNAME=<username> \
--build-arg REPOSITORY_NAME=<repo> \
--build-arg WEB_FILE_ZIP=<file.zip> \
--build-arg WEB_FILE_UNZIP=<file.zip> \
--build-arg DOMAIN_NAME=<domain> \
--build-arg RDS_ENDPOINT=<rds-endpoint> \
--build-arg RDS_DB_NAME=<db-name> \
--build-arg RDS_MASTER_USERNAME=<username> \
--build-arg RDS_DB_PASSWORD=<password> \
-t <image-tag> .
```

---

# Amazon ECR

Create an ECR repository:

```bash
aws ecr create-repository \
--repository-name <repository-name> \
--region <region>
```

Authenticate Docker:

```bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin \
<aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

Tag the image:

```bash
docker tag <image-tag> <repository-uri>
```

Push image:

```bash
docker push <repository-uri>
```

---

# ECS and Fargate Deployment

## IAM Roles

### ECS Task Execution Role

Used by ECS to:

* Pull images from ECR
* Write logs to CloudWatch

### ECS Task Role

Attach:

* AmazonS3ReadOnlyAccess
* SecretsManagerReadWrite
* AmazonSSMFullAccess
* CloudWatchLogsFullAccess

---

## Create Target Group

Configure a target group for ECS tasks.

---

## Create Application Load Balancer

Configure:

* HTTPS Listener (443)
* SSL Certificate from ACM
* Target Group Association

---

## Create ECS Cluster

Create an ECS Cluster using AWS Fargate.

---

## Create ECS Task Definition

Configure:

* Container Image from ECR
* CPU and Memory Allocation
* Port Mappings
* Task Execution Role

---

## Create ECS Service

Deploy ECS Tasks:

* Attach to Target Group
* Enable Auto Scaling
* Use Private Application Subnets

---

# Route 53 DNS

Create a DNS record pointing your domain name to the Application Load Balancer.

Example:

```text
www.example.com
        |
        v
Application Load Balancer
```

---

# Result

The application is deployed using a secure, scalable, and highly available AWS architecture featuring:

* Docker Containerization
* Amazon ECS with Fargate
* Amazon ECR
* Application Load Balancer
* Amazon RDS MySQL
* AWS Secrets Manager
* Route 53 DNS
* ACM SSL Certificates
* Multi-AZ Networking
* Private Application Infrastructure
* EC2 Instance Connect Endpoint for Administration

