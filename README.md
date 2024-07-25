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
- Launch an EC2 instance.
-  Choose Amazon Linux 2 AMI (HVM), SSD Volume Type.
- Select an instance type (t2.micro).
-  Configure instance details.
-  Add storage.
-  Configure security group to allow SSH (port 22) and application port ( docker : port 3000).
-  Review and launch.
-  Create a new key pair or use an existing one to connect to your instance.

#### Connect to Your EC2 Instance
-  Use EC2 Instance Connect or an SSH client to connect to your instance.

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

### Prepare Your Application

#### Create the Node.js Application
- Create a new directory for your project.
- Initialize a new Node.js application


#### Install Express.js:
npm install express

#### create an app.js file with the following content:
nano app.js

***code***

const express = require('express');

const app = express();

app.get('/', (req, res) => {

    res.send('Hello, World!');
    
});

const port = process.env.PORT || 3000;

app.listen(port, () => {

    console.log(`App running on port ${port}`);
    
});

### 2. Install Docker on EC2 Instance

sudo yum update -y

sudo amazon-linux-extras install docker

sudo service docker start

sudo usermod -a -G docker ec2-user

#### In the project directory, create a file named Dockerfile with the following content:
nano Dockerfile

// Use the official Node.js image.// 

FROM node:14

//  Create and change to the app directory.// 

WORKDIR /usr/src/app

//  Copy application dependency manifests to the container image.// 

COPY package*.json ./

//  Install dependencies.// 

RUN npm install

// Copy local code to the container image.// 

COPY . .

// Run the web service on container startup.// 

CMD [ "node", "app.js" ]

//  Document that the service listens on port 3000.// 

EXPOSE 3000

### 3. Create an IAM Role

- Go to the AWS Management Console.
- Navigate to the IAM Dashboard.
- Click on "Roles" in the left sidebar, then click "Create role".
- Choose "EC2" as the service that will use this role and click "Next: Permissions".
- Attach the following policy by searching for and selecting "AmazonEC2ContainerRegistryFullAccess".
- Click "Next: Tags" (you can skip adding tags) and then "Next: Review".
- Give your role a name (e.g., ECRAccessRole) and click "Create role".
- Attach the IAM Role to Your EC2 Instance
- Go to the EC2 Dashboard in the AWS Management Console.
- Select your running instance.
- Click on "Actions", then "Security", and then "Modify IAM role".
- Select the role you created (ECRAccessRole) from the dropdown and click "Update IAM role".

### 4. Build and Push Docker Image to Amazon ECR

#### Install AWS CLI:
sudo yum install aws-cli -y

#### Configure AWS CLI:
aws configure

#### Authenticate Docker to Your Amazon ECR Registry:

aws ecr get-login-password --region &lt;your-region&gt; | docker login --username AWS --password-stdin &lt;your-aws-account-id&gt;.dkr.ecr.&lt;your-region&gt;.amazonaws.com


#### Create an ECR Repository:
aws ecr create-repository --repository-name simple-web-app

#### Build your Docker image
sudo docker build -t my-app .

##### Tag the Docker image
sudo docker tag my-app:latest &lt;your-aws-account-id&gt;.dkr.ecr.&lt;your-region&gt;.amazonaws.com/my-app:latest

#### Push the Docker image to your ECR repository
sudo docker push &lt;your-aws-account-id&gt;.dkr.ecr.&lt;your-region&gt;.amazonaws.com/my-app:latest

#### Set Up Amazon ECS with Fargate

### 5. Create an ECS Cluster:

- Go to the ECS console.
- Click on "Clusters" and then "Create Cluster".
- Select "Networking only" for Fargate.
- Follow the steps to create the cluster.

### 5.  Create a Task Definition:

- In the ECS console, go to "Task Definitions".
- Click "Create new Task Definition".
- Select "Fargate".
- Configure the task definition:
- Task Definition Name: simple-web-app
- Container Definitions: Add a container with the following settings:
- Container Name: simple-web-app
- Image: <your-aws-account-id>.dkr.ecr.<your-region>.amazonaws.com/simple-web-app:latest
- Memory Limits: 512 (soft limit)
- Port Mappings: 3000 (host) and 3000 (container)
- Task Role: ecsTaskExecutionRole (ensure this IAM role exists)
- Task Size:
- Task Memory: 1GB
- Task CPU: 0.5 vCPU

### 6. Create a Service:

- In the ECS console, go to "Clusters".
- Select your cluster.
- Click "Create" and select "Create Service".
- Select "Fargate" as the launch type.
- Configure the service with the task definition you created, desired number of tasks, VPC, and subnets.

### 7. Verify and Test
Check ECS Tasks: Ensure that your ECS tasks are running correctly by checking the ECS console.

### 8. Commands used in the whole project for debugging

- Verify the Image: sudo docker images
- Run the Container: sudo docker run -d -p 80:80 new-project
- Check Container Status: sudo docker ps
- View Logs: sudo docker logs <container_id>
- Check Docker Service Status: sudo systemctl status docker
- Start Docker Service: sudo systemctl start docker
- Restart Docker: sudo systemctl restart docker
- Run Commands with Sudo: sudo docker build -t new-project
