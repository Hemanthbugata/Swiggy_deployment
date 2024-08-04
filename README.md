# Project Title 
<h2>Securing Swiggy Clone App Deployment on AWS: Building a Robust DevSecOps Pipeline with Terraform, Jenkins, SonarQube, Trivy, ArgoCD, and EKS</h2>

# Project Description 
<h2>This project aims to create a secure and scalable deployment pipeline for a Swiggy Clone application on AWS, leveraging the power of modern DevSecOps tools and practices. The pipeline integrates Terraform, Jenkins, SonarQube, Trivy, ArgoCD, and Amazon EKS to ensure continuous integration, continuous deployment, and robust security throughout the development lifecycle.</h2>

<h3>Key Components:</h3>

Terraform: Infrastructure as Code (IaC) tool to provision and manage AWS resources, ensuring consistent and repeatable deployments.

Jenkins: Automation server for building, testing, and deploying the application, incorporating CI/CD best practices.

SonarQube: Static code analysis tool to identify code quality issues and security vulnerabilities early in the development process.

Trivy: Vulnerability scanner to ensure container images are free from known security issues before deployment.

ArgoCD: GitOps continuous delivery tool for Kubernetes, managing application deployments and promoting a declarative approach to CI/CD.

Amazon EKS: Managed Kubernetes service to run and scale the Swiggy Clone application with high availability and security.

<h3>Objectives</h3>

Infrastructure Provisioning: Use Terraform to create a robust AWS infrastructure, including VPC, subnets, security groups, EKS cluster, and necessary IAM roles.

CI/CD Pipeline: Configure Jenkins to automate the build, test, and deployment processes, integrating SonarQube for static code analysis.

Security Scanning: Implement Trivy to scan container images for vulnerabilities before they are pushed to the container registry.

GitOps Deployment: Utilize ArgoCD to manage Kubernetes deployments, ensuring that the application state matches the desired configuration stored in Git.

Monitoring and Logging: Set up comprehensive monitoring and logging to track application performance and security incidents.

Documentation and Best Practices: Provide detailed documentation and adhere to DevSecOps best practices to ensure the pipeline is maintainable and secure.

Create an IAM user with administration access.

Log in to the AWS Console with the above user.

Create one free-tier EC2 instance with Ubuntu. (AWSCLI-INSTANCE)

Step 1:

  install AWS CLI: 

  ```
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install
  aws --version
```
```
aws configure
```
 ( aws access key and aws secret key, can find in IAM user (security credentials) , create a access key )

 Step 2 :

   Terraform installation in AWS-CLI INSTANCE : (connect to the instance)

   ```
    wget https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_linux_amd64.zip
    unzip terraform_1.3.7_linux_amd64.zip
    mv terraform /usr/local/bin
    sudo mv terraform /usr/local/bin
    terraform -v
```
here have to setup jenkins, sonarqube, trivy new instance for CI/CD via terraform files .

Main.tf 

```
resource "aws_instance" "web" {
  ami                    = "ami-0fc5d935ebf8bc3bc"   #change ami id for different region
  instance_type          = "t2.large"
  key_name               = "Swiggy-access"
  vpc_security_group_ids = [aws_security_group.Jenkins-sg.id]
  user_data              = templatefile("./install.sh", {})

  tags = {
    Name = "Jenkins-sonarqube-trivy-vm"
  }

  root_block_device {
    volume_size = 30
  }
}

resource "aws_security_group" "Jenkins-sg" {
  name        = "Jenkins-sg"
  description = "Allow TLS inbound traffic"

  ingress = [
    for port in [22, 80, 443, 8080, 9000, 3000] : {
      description      = "inbound rules"
      from_port        = port
      to_port          = port
      protocol         = "tcp"
      cidr_blocks      = ["0.0.0.0/0"]
      ipv6_cidr_blocks = []
      prefix_list_ids  = []
      security_groups  = []
      self             = false
    }
  ]

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "jenkins-sg"
  }
}
```
Provider.tf
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"  #change your region
}
```
Install.sh 
```
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

#install docker
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu  
newgrp docker
sudo chmod 777 /var/run/docker.sock
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

#install trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```
after saving these 3 files under a directory (swiggy-deploy)

Exexcution of Terraform commands :
  terraform init
  terraform plan
  terraform validate
  terraform apply -auto-approve

Step 3:
Now take the Public ip of new instance generated by terraform scripts with 8080 to access jenkins.

With same publicip:9000 access the sonarqube for code analysis and quality gates.(login:admin password : admin)

Jenkins setup :

Install plugins : 

1. Eclipse Temurin Installer (Install without restart)
2. SonarQube Scanner (Install without restart)
3. NodeJs Plugin (Install without restart)
4. Sonar Quality Gates (Install without restart)
5. OWASP Dependency Check (Install without restart)
6. Docker (Install without restart)
7. Docker Commons (Install without restart)
8. Docker Pipeline (Install without restart)
9. Docker API (Install without restart)
10. docker-build-step (Install without restart)
11. stage-view pipeline  (Install without restart)

Step 4:

  configuration of java and nodejs (Manage jenkins --> Tools)

     ![image](https://github.com/user-attachments/assets/1a07bfba-d350-4186-b5f9-06403ceac4cc)

     ![image](https://github.com/user-attachments/assets/5835aa81-db6b-488a-8eba-d28ca552f5b5)

Step 5 :

    Sonarqube 


