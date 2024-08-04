# Project Title 
<h1>Securing Swiggy Clone App Deployment on AWS: Building a Robust DevSecOps Pipeline with Terraform, Jenkins, SonarQube, Trivy, ArgoCD, and EKS</h1>

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
