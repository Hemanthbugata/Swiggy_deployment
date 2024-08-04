<h2> Project Title</h2>
<h2>Securing Swiggy Clone App Deployment on AWS: Building a Robust DevSecOps Pipeline with Terraform, Jenkins, SonarQube, Trivy, ArgoCD, and EKS</h2>

<h2>Project Description</h2>
<h2>This project aims to create a secure and scalable deployment pipeline for a Swiggy Clone application on AWS, leveraging the power of modern DevSecOps tools and practices. The pipeline integrates Terraform, Jenkins, SonarQube, Trivy, ArgoCD, and Amazon EKS to ensure continuous integration, continuous deployment, and robust security throughout the development lifecycle.</h2>

<h2>Project Architecture</h2>

![_Minimalist Elegant Flowchart Brainstorm Project Mind Map (2)](https://github.com/user-attachments/assets/f5113742-6ae1-4807-a61a-38775eaac535)

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

  Goto Manage Jenkins → Tools → Install JDK (17) and NodeJs (16). Click on Apply and Save

  configuration of java and nodejs (Manage jenkins --> Tools)


5. Configure Sonar Server in Manage Jenkins ⚙️

Grab the public IP address of your EC2 instance.

    public IP>:9000.

Sonarqube works on Port 9000, 

Go to your Sonarqube server.

Click on Administration → Security → Users → Click on Tokens and Update Token, → Give it a name, and click on Generate Token


click on update Token


Create a token with a name and generate



copy Token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add secret text. It should look like this



You will see this page once you click on create


Now, go to Dashboard → Manage Jenkins → System and add like the below image.


Click on Apply and Save.

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.


In the Sonarqube Dashboard, add a quality gate as well.

In the sonar interface, create the quality gate as shown below:

Click on the quality gate, then create.



Click on the save option.

In the Sonarqube Dashboard, Create Webhook option as shown in below:

Administration → Configuration →Webhooks


Click on Create


Add details:

<http://jenkins-private-ip:8080>/sonarqube-webhook/
Let’s go to our pipeline and add the script to our pipeline script.
```
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Hemanthbugata/Swiggy_deployment.git'
            }
        }
        stage("Sonarqube Analysis ") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy-CICD \
                    -Dsonar.projectKey=Swiggy-CICD '''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}

```
Click on Build now, and you will see the stage view like this:


To see the report, you can go to Sonarqube Server and go to Projects.


You can see the report has been generated, and the status shows as passed. You can see that there are 765 lines it has scanned. To see a detailed report, you can go to issues.


6. Install OWASP Dependency Check Plugins 🧐
Go to Dashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restarting.


First, we configured the plugin, and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools →


Click on Apply and save here.

Now go to Configure → Pipeline and add this stage to your pipeline and build.
```
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```
You will see that in status, a graph will also be generated for vulnerabilities.


7. Docker Image Build and Push 🐳
We need to install the Docker tool on our system.

Go to Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins.


Now, goto Dashboard → Manage Jenkins → Tools →


Now go to the Dockerhub repository to generate a token and integrate with Jenkins to push the image to the specific repository.


If you observe, there is no repository related to Swiggy.

There is an icon with the first letter of your name.

Click on that My Account, → Settings → Create a new token and copy the token.





Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add secret text. It should look like this:


Add this stage to Pipeline Script.
```
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){ 
                    app.push("${env.BUILD_NUMBER}")  
                       sh "docker build -t swiggy-app ."
                       sh "docker tag swiggy-app hemanth0102/swiggy-app:latest "
                       sh "docker push hemanth0102/swiggy-app:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image hemanth0102/swiggy-app:latest > trivyimage.txt" 
            }
        }
```
You will be able to view the output in the Jenkins pipeline and output upon successful execution.





When you log in to Dockerhub, you will see a new image is created.



8. Creation of EKS Cluster with ArgoCD 🌐
EKSCTL Installation:

Now let’s install EKSCTL in Ubuntu EC2 which was created earlier.
```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```
# Check the eksctl version.
eksctl version

Command to Create EKS Cluster using eksctl command:

eksctl create cluster --name <name-of-cluster> --nodegroup-name <nodegrpname> --node-type <instance-type> --nodes <no-of-nodes>

```
eksctl create cluster --name swiggy-eks-cluster --nodegroup-name ng-test --node-type t3.medium --nodes 2
```

It will take 5–10 minutes to create a cluster.


As you will see in the EC2 instances running list one instance is running in the name of EKS Cluster as shown below.


EKS Cluster is up and ready and check with the below command.

```
Now let’s install ArgoCD in the EKS Cluster.

kubectl create ns Argocd
# This will create a new namespace, argocd, where Argo CD services and application resources will live.
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Download Argo CD CLI:
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```
Access The Argo CD API Server:

# By default, the Argo CD API server is not exposed with an external IP. To access the API server, 
choose one of the following techniques to expose the Argo CD API server:
* Service Type Load Balancer
* Port Forwarding
Let’s go with Service Type Load Balancer.

```
# Change the argocd-server service type to LoadBalancer.
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
```
List the resources in the namespace:

kubectl get all -n argocd

Get the load balancer URL:

kubectl get svc -n argocd
```

Pickup the URL and paste it into the web to get the UI as shown below image:


Login Using The CLI:

argocd admin initial-password -n argocd

Login with the admin and Password in the above you will get an interface as shown below:


Click on New App:


Enter the Repository URL, set path to ./, Cluster URL to kubernetes.default.svc, the namespace to default and click save.

The GitHub URL is the Kubernetes Manifest files which I have stored and the pushed image is used in the Kubernetes deployment files.

Repo Link: 

https://github.com/Hemanthbugata/Swiggy_deployment.git

You should see the below, once you’re done with the details.


Click on it.


You can see the pods running in the EKS Cluster.


We can see the out-of-pods using the load balancer URL:

 ```
kubectl get svc
```

With the above load balancer, you will be able to see the output as shown in the below image:


As you observe in the above image, I just want to change the address of the Swiggy Application.

Then the real magic will happen, the changes updated in the code we will try to push the code to GitHub and run a pipeline to push the image to the repository with the updated details.


I will click on the sync option which is in the ArgoCD and then the updates will be in our Swiggy Website.



![argo cd deployment](https://github.com/user-attachments/assets/2b86627a-1b17-4483-8965-4a3268c24bcc)


![address change swiggy](https://github.com/user-attachments/assets/9ea49cc5-c417-460d-b806-0e04805f82c4)

