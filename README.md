# SWE 645 - Assignment 2

Ryan Brooks  
SWE 645  
Dr. Dubey

## Overview

This assignment containerizes the Assignment 1 static website, deploys it to Kubernetes on EC2, and automates build/push/deploy with Jenkins.

Project components:
- Docker image: `ryanline/simple-webapp:latest`
- Kubernetes manifests: `k8s/deployment.yaml`, `k8s/service.yaml`
- Jenkins pipeline: `Jenkinsfile`

## Installation and Setup

### Prerequisites

- AWS EC2 Ubuntu instance
- Docker installed on the host/Jenkins node
- Kubernetes cluster access from the host (`kubectl` configured)
- Jenkins installed with Docker and Pipeline support
- Docker Hub account and credentials stored in Jenkins (`docker-pass`)
- Git installed

### Local Project Setup

```bash
git clone https://github.com/Ryanline/swe645test.git
cd swe645test
```

### Build and Push Docker Image Manually (Optional Verification)

```bash
docker build -t ryanline/simple-webapp:latest .
docker login
docker push ryanline/simple-webapp:latest
```

## Kubernetes Deployment Details

Kubernetes files are located in `k8s/`.

- `k8s/deployment.yaml`
  - Creates `simple-webapp-deployment`
  - Uses image `ryanline/simple-webapp:latest`
  - Runs 3 replicas
  - Exposes container port `80`
- `k8s/service.yaml`
  - Creates NodePort service `my-service`
  - Maps service port `80` to target port `80`
  - Exposes NodePort `30007`

### Deploy to Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### Validate Deployment

```bash
kubectl get deployments
kubectl get pods -o wide
kubectl get svc
kubectl rollout status deployment/simple-webapp-deployment
```

Access pattern:
- `http://<EC2_PUBLIC_IP>:30007`

Security group note:
- Ensure inbound TCP `30007` is allowed (or NodePort range `30000-32767`).

## CI/CD Pipeline Details (Jenkins)

The `Jenkinsfile` defines a pipeline with these stages:

1. `Checkout`
   - Pulls branch `main` from `https://github.com/Ryanline/swe645test.git`
2. `Build Docker Image`
   - Builds `ryanline/simple-webapp:latest`
3. `Push to Docker Hub`
   - Pushes the image to Docker Hub using Jenkins credential `docker-pass`
4. `Deploy to Kubernetes`
   - Updates deployment image with:
     - `kubectl set image deployment/simple-webapp-deployment simple-webapp=ryanline/simple-webapp:latest`
   - Waits for rollout completion with:
     - `kubectl rollout status deployment/simple-webapp-deployment`

Pipeline outcome:
- On successful runs, Kubernetes is updated to the latest container image version.
- On failure, Jenkins marks the build failed and deployment is not considered complete.


# SWE 645 – Assignment 1

## URLs:

### Amazon S3 Static Website
- Homepage:  
  http://swe-645-assignment1-ryanbrooks.s3-website.us-east-2.amazonaws.com/index.html
- Survey Page:  
  http://swe-645-assignment1-ryanbrooks.s3-website.us-east-2.amazonaws.com/survey.html

### Amazon EC2 Web Server
- Homepage:  
  http://18.216.119.163/
- Survey Page:  
  http://18.216.119.163/survey.html

## Part 1: Deploying the Website on Amazon S3

The following steps describe how to deploy the website using Amazon S3 static website hosting.

### Steps

1. Log into the **AWS Management Console** and navigate to **Amazon S3**.
2. Create a new S3 bucket:
   - Bucket name must be globally unique (e.g., `swe-645-assignment1-yourname`)
   - Region: `us-east-2`
3. Disable **Block Public Access** for the bucket and acknowledge the warning.
4. Enable **Static Website Hosting**:
   - Index document: `index.html`
   - Error document: `error.html`
5. Upload the following files to the bucket:
   - `index.html`
   - `survey.html`
   - `error.html`
6. Add the following bucket policy to allow public read access (replace `YOUR-BUCKET-NAME` with your bucket name):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```
7. Save the bucket policy and verify the website is accessible using the S3 static website endpoint.

## Part 2: Deploying the Website on Amazon EC2

The following steps describe how to deploy the same website on an EC2 instance running Ubuntu and Apache.

### Steps

1. Log into the **AWS Management Console** and navigate to **EC2**.
2. Launch a new EC2 instance:
   - AMI: **Ubuntu Server 24.04 LTS**
   - Instance type: **t3.micro**
   - Auto-assign public IPv4 address: **Enabled**
3. Configure the security group:
   - Allow **SSH (TCP 22)** from your IP address
   - Allow **HTTP (TCP 80)** from anywhere (`0.0.0.0/0`)
4. Launch the instance and connect using **EC2 Instance Connect**.
5. Update the system and install Apache:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

6. Navigate to the Apache web root directory:

```bash
cd /var/www/html
```

7. Remove the default Apache index file:

```bash
sudo rm index.html
```

8. Download the website files from the S3 static website:

```bash
sudo curl -O http://swe-645-assignment1-ryanbrooks.s3-website.us-east-2.amazonaws.com/index.html
sudo curl -O http://swe-645-assignment1-ryanbrooks.s3-website.us-east-2.amazonaws.com/survey.html
sudo curl -O http://swe-645-assignment1-ryanbrooks.s3-website.us-east-2.amazonaws.com/error.html
```

9. Set appropriate permissions for the files:

```bash
sudo chmod 644 *.html
```

10. Verify the deployment by accessing the EC2 public IP address in a web browser.
