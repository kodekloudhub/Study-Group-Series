# Challenge 2: Create an ECS Service with Nginx Reverse Proxy with Terraform

In this challenge, you will set up an AWS ECS service that runs an Nginx server configured to forward requests to another application in the same cluster with Terraform. You will then use a jumphost to verify that the Nginx service is forwarding requests correctly.

**Prerequisites**:
1. AWS account
2. Terraform installed locally

## 1. Define the VPC and Subnets
- Create a VPC to host your ECS services.
- Define private subnets where your ECS tasks will run.

## 2. Create Security Groups
- Set up security groups to control traffic to and from your ECS services and jumphost.

## 3. Create ECS Cluster
- Define and create an ECS cluster to host your services.

## 4. Create Backend Application Task Definition
- Create a task definition for your backend application that will receive requests from Nginx. You can use kodekloud/simple-webapp image

## 5. Create Nginx Task Definition with Reverse Proxy Configuration
- Configure Nginx as a reverse proxy to forward requests to the backend application.
- Build and push the Nginx Docker image with the necessary configuration to a Docker registry.

## 6. Create ECS Services
- Deploy the backend application as an ECS service.
- Deploy Nginx as a separate ECS service in the same cluster.

## 7. Set Up a Jumphost
- Create a jumphost in a public subnet to access the private ECS services.
- Ensure the jumphost has the necessary permissions and network access to connect to the ECS services.

## 8. Verify Nginx Service via Jumphost
- SSH into the jumphost.
- Use tools like `curl` to verify that the Nginx service is correctly forwarding requests to the backend application.