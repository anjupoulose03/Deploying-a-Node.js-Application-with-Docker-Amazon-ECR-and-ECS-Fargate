# Deploying a Node.js Application with Docker, Amazon ECR, and ECS Fargate

This project demonstrates how to deploy a Node.js application using Docker, Amazon ECR, and ECS Fargate. The application logs were monitored using Amazon CloudWatch.

## Prerequisites
- AWS Account
- IAM Role with necessary permissions for ECR, ECS, and CloudWatch
- EC2 Instance with Amazon Linux 2
- Docker installed on EC2
- AWS CLI installed on EC2

## Steps

Create a Simple Web Application
Containerize the Application with Docker
Push the Docker Image to ECR
Create an ECS Cluster
Create a Task Definition
Create a Service
Access the Deployed Application

### 1. Setting Up the Environment

#### Create an EC2 Instance
1. Launch an EC2 instance.
2. Choose Amazon Linux 2 AMI (HVM), SSD Volume Type.
3. Select an instance type (t2.micro).
4. Configure instance details.
5. Add storage.
6. Configure security group to allow SSH (port 22) and application port ( docker : port 3000).
7. Review and launch.
9. Create a new key pair or use an existing one to connect to your instance.

#### Connect to Your EC2 Instance
1. Use EC2 Instance Connect or an SSH client to connect to your instance.

#### Update Package Index:
sudo yum update -y

#### Install Node.js and npm:
curl -sL https://rpm.nodesource.com/setup_16.x | sudo bash -
sudo yum install -y nodejs

#### Verify Installation:
node -v
npm -v

#### Create a directory and initialize npm
mkdir my-project
cd my-project
npm init -y

#### Prepare Your Application

#### Create the Node.js Application
- Create a new directory for your project.
- Initialize a new Node.js application


#### Install Express.js:
npm install express

Create an app.js file with the following content:
nano app.js

const express = require('express');
const app = express();
const port = 3000;
app.get('/', (req, res) => res.send('Hello, World!'));
app.listen(port, () => console.log(`App running on port ${port}`));

Install Express.js:

npm install express

### 2. Install Docker on EC2 Instance

```bash
sudo yum update -y
sudo amazon-linux-extras install docker
sudo service docker start
sudo usermod -a -G docker ec2-user

#### Create a Dockerfile

In the project directory, create a file named Dockerfile with the following content:
nano Dockerfile

# Use the official Node.js image.
FROM node:14

# Create and change to the app directory.
WORKDIR /usr/src/app

# Copy application dependency manifests to the container image.
COPY package*.json ./

# Install dependencies.
RUN npm install

# Copy local code to the container image.
COPY . .

# Run the web service on container startup.
CMD [ "node", "app.js" ]

# Document that the service listens on port 3000.
EXPOSE 3000

Build and Push Docker Image to Amazon ECR
Install AWS CLI:
sudo yum install aws-cli -y
Configure AWS CLI:
aws configure

Authenticate Docker to Your Amazon ECR Registry:
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-aws-account-id>.dkr.ecr.<your-region>.amazonaws.com

Create an ECR Repository:
aws ecr create-repository --repository-name simple-web-app

Build Your Docker Image:
docker build -t simple-web-app .

Tag Your Docker Image:
docker tag simple-web-app:latest <your-aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/simple-web-app:latest

Push the Docker Image to ECR:
docker push <your-aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/simple-web-app:latest

Set Up Amazon ECS with Fargate
Create an ECS Cluster:

Go to the ECS console.
Click on "Clusters" and then "Create Cluster".
Select "Networking only" for Fargate.
Follow the steps to create the cluster.
Create a Task Definition:

In the ECS console, go to "Task Definitions".
Click "Create new Task Definition".
Select "Fargate".
Configure the task definition:
Task Definition Name: simple-web-app
Container Definitions: Add a container with the following settings:
Container Name: simple-web-app
Image: <your-aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/simple-web-app:latest
Memory Limits: 512 (soft limit)
Port Mappings: 3000 (host) and 3000 (container)
Task Role: ecsTaskExecutionRole (ensure this IAM role exists)
Task Size:
Task Memory: 1GB
Task CPU: 0.5 vCPU
Create a Service:

In the ECS console, go to "Clusters".
Select your cluster.
Click "Create" and select "Create Service".
Select "Fargate" as the launch type.
Configure the service with the task definition you created, desired number of tasks, VPC, and subnets.

#### Verify and Test
Check ECS Tasks: Ensure that your ECS tasks are running correctly by checking the ECS console.

#### Commands used in the process

Verify the Image: sudo docker images
Run the Container: sudo docker run -d -p 80:80 new-project
Check Container Status: sudo docker ps
View Logs: sudo docker logs <container_id>
Check Docker Service Status: sudo systemctl status docker
Start Docker Service: sudo systemctl start docker
Restart Docker: sudo systemctl restart docker
Run Commands with Sudo: sudo docker build -t new-project
