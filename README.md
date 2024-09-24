# Node.js To-Do List App with CI/CD on AWS EC2

## Project Overview

This project is a simple Node.js-based To-Do List application that demonstrates the use of Continuous Integration and Continuous Deployment (CI/CD) practices using Jenkins. The application is deployed on an AWS EC2 instance.

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Local Setup](#local-setup)
- [CI/CD Pipeline Setup](#cicd-pipeline-setup)
- [Deploying to AWS EC2](#deploying-to-aws-ec2)
- [Jenkins Configuration](#jenkins-configuration)
- [Screenshots](#screenshots)

## Features

- **Node.js Backend:** Implements a basic CRUD application for managing tasks in a to-do list.
- **EJS Templating:** Uses Embedded JavaScript (EJS) for rendering HTML templates.
- **Docker Support:** Dockerfile and `docker-compose.yaml` are included for containerization.
- **CI/CD Pipeline:** Jenkins is used to automate the testing, building, and deployment process.
- **AWS EC2 Deployment:** The application is deployed on an AWS EC2 instance using Jenkins.

## Tech Stack

- **Backend:** Node.js, Express.js
- **Templating Engine:** EJS
- **CI/CD Tool:** Jenkins
- **Deployment Platform:** AWS EC2
- **Containerization:** Docker

## Prerequisites

Before setting up the project, ensure you have the following tools installed:

- **Node.js and npm:** For running the Node.js application locally.
- **Docker:** For containerizing the application.
- **AWS CLI:** To interact with AWS services.
- **Jenkins:** For setting up CI/CD.

## Local Setup

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/your-github-username/your-repo-name.git
   cd your-repo-name
   ```

2. **Install Dependencies:**
   ```bash
   npm install
   ```

3. **Run the Application:**
   ```bash
   node app.js
   ```
   The app will be running at `http://localhost:3000`.

4. **Run the Application with Docker:**
   ```bash
   docker-compose up --build
   ```
   This will build and run the app in a Docker container.

## CI/CD Pipeline Setup

### 1. Set Up Jenkins

- **Install Jenkins:** Follow the official [Jenkins installation guide](https://www.jenkins.io/doc/book/installing/).
- **Install Required Plugins:**
  - NodeJS Plugin
  - Git Plugin
  - Docker Pipeline Plugin

### 2. Configure Jenkins Job

1. **Create a New Job:**
   - Open Jenkins dashboard.
   - Click on **New Item**, select **Pipeline**, and give it a name.

2. **Pipeline Configuration:**
   - **Source Code Management:** Set to Git and enter your repository URL.
   - **Build Triggers:** Set up to build on Git commit.
   - **Pipeline Script:** Add the following Jenkinsfile script:
     ```groovy
     pipeline {
         agent any
         tools {
             nodejs "NodeJS"
         }
         stages {
             stage('Clone Repository') {
                 steps {
                     git url: 'https://github.com/your-github-username/your-repo-name.git'
                 }
             }
             stage('Install Dependencies') {
                 steps {
                     sh 'npm install'
                 }
             }
             stage('Run Tests') {
                 steps {
                     sh 'npm test'
                 }
             }
             stage('Build Docker Image') {
                 steps {
                     sh 'docker build -t todo-app .'
                 }
             }
             stage('Deploy to EC2') {
                 steps {
                     sshagent(['ec2-ssh-key']) {
                         sh 'scp -o StrictHostKeyChecking=no docker-compose.yaml ec2-user@<EC2_PUBLIC_IP>:/home/ec2-user/'
                         sh 'ssh ec2-user@<EC2_PUBLIC_IP> docker-compose up -d'
                     }
                 }
             }
         }
     }
     ```

3. **Set Up SSH Key for EC2:**
   - Generate an SSH key and add it to your Jenkins credentials.
   - Upload the public key to your EC2 instance.

### 3. Deploying to AWS EC2

1. **Launch an EC2 Instance:**
   - Go to the AWS Management Console and launch a new EC2 instance with Ubuntu or Amazon Linux.
   - Open necessary ports (e.g., 22 for SSH, 80 for HTTP) in the security group.

2. **Install Docker on EC2:**
   SSH into your EC2 instance:
   ```bash
   sudo apt-get update
   sudo apt-get install docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

3. **Run the Application:**
   - The Jenkins pipeline will automatically SSH into the EC2 instance, copy the `docker-compose.yaml` file, and run the application using Docker.



## Troubleshooting

### Common Issues and Solutions

- **EC2 SSH Connection Issues:**
  - Ensure that your EC2 security group allows inbound SSH connections on port 22.
  - Verify that the correct private key is used for the SSH connection.
  
- **Jenkins SSHAgent Plugin Errors:**
  - Double-check that the SSH credentials in Jenkins are correctly configured and linked to your EC2 key pair.
  - Ensure the SSH key has appropriate permissions set.

- **Docker Permission Denied on EC2:**
  - If you encounter a "permission denied" error while running Docker commands on EC2, ensure that the user is added to the Docker group:
    ```bash
    sudo usermod -aG docker $USER
    ```

- **Application Not Accessible via Browser:**
  - Verify that port 80 is open in your EC2 instance's security group.
  - Ensure that the application is running without errors on the EC2 instance.

## Future Enhancements

- **Automated Testing:** Integrate more comprehensive unit and integration tests into the CI/CD pipeline.
- **Load Balancing:** Implement an AWS Elastic Load Balancer (ELB) to distribute traffic across multiple instances.
- **Database Integration:** Connect the app to a managed database service like AWS RDS for persistent data storage.
- **Auto-Scaling:** Set up auto-scaling groups on AWS to handle varying traffic loads.

## Conclusion

This project demonstrates the full lifecycle of a Node.js application from development to deployment using modern DevOps practices. By leveraging Jenkins for CI/CD and AWS EC2 for deployment, you can automate the entire process, ensuring consistent and reliable application delivery.

This setup is a robust foundation for further enhancements, such as scaling the application, adding more sophisticated monitoring and alerting, and integrating with other AWS services.

