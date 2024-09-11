Certainly! Here’s a detailed README for your project based on the steps and instructions you’ve provided:

---

# Web Application Deployment Using Docker, ECS, and ECR

## Overview

This project involves deploying a web application using Docker containers, managed by AWS ECS (Elastic Container Service) with the Fargate launch type, and stored in AWS ECR (Elastic Container Registry). The infrastructure includes a 3-tier VPC setup, NAT gateway, security groups, and RDS instances. The application is deployed with secure access and scalable management.

![image](https://github.com/user-attachments/assets/2b9ab5c8-9270-4d8f-9af6-db049cf41432)


## Project Structure

- **Dockerfile**: Containerizes the web application.
- **ecs-task-definition.json**: Defines the task configuration for ECS.
- **Docker Image**: Stored in AWS ECR.
- **AWS Infrastructure**: Includes VPC, subnets, NAT gateways, security groups, and RDS instances.
- **Flyway**: For database migration.

## Prerequisites

- **GitHub Account**: To store application code and Dockerfile.
- **Key Pairs**: For cloning the GitHub repository and SSH access.
- **Git**: Version control system.
- **Visual Studio Code**: For editing scripts and Dockerfile.
- **Docker**: To build and manage container images.
- **Docker Hub Account**: Optional, for image storage.
- **Flyway**: For database migrations.
- **AWS CLI**: For interacting with AWS services from the command line.

## AWS Infrastructure Setup (3-Tier VPC)

1. **Create VPC**:
   - Set up a Virtual Private Cloud (VPC) with appropriate CIDR block.

2. **Internet Gateway**:
   - Create an Internet Gateway and attach it to your VPC to allow internet access.

3. **Availability Zones**:
   - Create two Availability Zones (AZs) for high availability.

4. **Subnets**:
   - **Public Subnets**: One in each AZ for resources like Bastion Host, NAT Gateway, and Application Load Balancer (ALB).
   - **Private Subnets**: One in each AZ for Web Servers and RDS instances.

5. **Route Tables**:
   - **Public Route Table**: Connects to the Internet Gateway and is associated with public subnets.
   - **Private Route Tables**: Associated with private subnets, routes traffic to the internet via NAT Gateways.

6. **NAT Gateways**:
   - Set up NAT Gateways in each public subnet for internet access from private subnets.

7. **Security Groups**:
   - **ALB**: For traffic from the internet.
   - **Bastion Host**: For SSH access from your IP address.
   - **Container Service (ECS Tasks)**: For traffic from the ALB.
   - **Database Instance**: For traffic from Bastion Host or Containers.

8. **RDS Instances**:
   - Create RDS instances in each AZ with subnet groups for high availability.

9. **Domain Registration**:
   - Register a domain name in Route 53 for your web application.

## Build and Deploy Docker Image

1. **Build Docker Image**:
   - Create a `Dockerfile` in your project directory.
   - Build the Docker image using the script below:

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

   - Make the script executable and run it using PowerShell:

     ```powershell
     Set-ExecutionPolicy -ExecutionPolicy Unrestricted
     .\path\to\script.ps1
     ```

2. **Elastic Container Registry (ECR)**:
   - Create an ECR repository:

     ```bash
     aws ecr create-repository --repository-name <repository-name> --region <region>
     ```

   - Authenticate Docker to ECR and push the Docker image:

     ```bash
     docker tag <image-tag> <repository-uri>
     aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
     docker push <repository-uri>
     ```

## Database Migration

1. **SSH Access**:
   - Create a key pair for SSH access.
   - Set up a Bastion Host in AWS for SSH tunneling.

2. **Migrate Database**:
   - Set up SSH tunnel from local machine to Bastion Host:

     ```bash
     ssh -i <key_pair.pem> ec2-user@<bastion-public-ip> -L 3306:<rds-endpoint>:3306 -N
     ```

   - Run Flyway migrations:

     ```bash
     flyway migrate
     ```

## Application Load Balancer (ALB) and HTTPS

1. **Create ALB**:
   - Set up an Application Load Balancer to route traffic to ECS tasks.

2. **SSL Certificate**:
   - Request an SSL certificate in AWS Certificate Manager (ACM).
   - Add an HTTPS listener (port 443) to ALB and associate the SSL certificate.

3. **Environment File**:
   - Create an environment file for your application settings.
   - Upload the file to an S3 bucket.

## ECS Tasks Setup

1. **Create ECS Cluster**:
   - Set up an ECS cluster using the ECS Management Console or AWS CLI.

2. **Create Task Definition**:
   - Define the task configuration in `ecs-task-definition.json` and register it:

     ```bash
     aws ecs register-task-definition --cli-input-json file://ecs-task-definition.json
     ```

3. **Create ECS Service**:
   - Create a service within the ECS cluster to manage task instances.
   - Configure subnets, security groups, ALB, and set desired task count.

4. **Route 53 Record Set**:
   - Create a record set in Route 53 for your domain:
     - Name: `www`
     - Type: `A`
     - Value: ECS service's ALB DNS name.

## Additional Notes

- **Scaling**: Configure auto-scaling policies for ECS tasks and RDS instances.
- **Security**: Implement best practices for securing your VPC, ECS tasks, and RDS instances.

## References

- [AWS ECS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-amazon-ecs.html)
- [AWS ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [Docker Documentation](https://docs.docker.com/)
- [Flyway Documentation](https://flywaydb.org/documentation/)

## License

Specify the license under which your project is distributed, if applicable.

---

Feel free to adjust this README to match your exact setup and additional project-specific details.
