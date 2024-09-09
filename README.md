# CI/CD Pipeline (Deliverable 2)

This CI/CD pipeline, defined in GitHub Actions, automates the process of deploying a regtech application to an EKS cluster. It performs the following steps:

Gitleaks Scan: Scans the codebase for sensitive information using Gitleaks.
SonarQube Analysis: Runs code quality and security analysis using SonarQube.
Docker Build, Scan, and Push:
Builds a Docker image of the application.
Scans the image for vulnerabilities using Trivy.
Pushes the image to AWS Elastic Container Registry (ECR).
Deploy to EKS with ArgoCD:
Logs in to ArgoCD and syncs the application to the EKS cluster.
Updates the Kubernetes deployment manifest with the latest Docker image.
Monitors the deployment status and performs a rollback if the deployment fails.

## Environment Variables to be Set
These variables are required for the CI/CD pipeline to function properly:

### AWS Credentials and Configuration:
AWS_REGION: AWS region for ECR and EKS.
AWS_ACCESS_KEY_ID: AWS access key with permissions for ECR and EKS.
AWS_SECRET_ACCESS_KEY: Secret key for the above AWS access key.
ECR Repository Details wich include
ECR_REGISTRY: URI of the AWS ECR registry.
ECR_REPOSITORY: Name of the AWS ECR repository.

### SonarQube Configuration wich include:
SONAR_TOKEN: Authentication token for SonarQube.
SONAR_HOST_URL: URL of the SonarQube server.

### ArgoCD Configuration:
ARGOCD_SERVER: URL of the ArgoCD server.
ARGOCD_USERNAME: Username for ArgoCD CLI login.
ARGOCD_PASSWORD: Password for ArgoCD CLI login.

## Docker Image Tag:
IMAGE_TAG: The tag of the Docker image (set dynamically in the pipeline using Git commit SHA).

### Where to Set the Variables

GitHub Secrets: Store sensitive variables in your repository under Settings > Secrets and variables > Actions > New repository secret.
Sensitive Variables are: AWS_REGION, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, ECR_REGISTRY, ECR_REPOSITORY, SONAR_TOKEN, SONAR_HOST_URL, ARGOCD_SERVER, ARGOCD_USERNAME, ARGOCD_PASSWORD.

GitHub Actions Environment Variables: Set dynamically within the workflow file (.github/workflows/ci.yml).
Example: IMAGE_TAG is set using echo "IMAGE_TAG=${GITHUB_SHA}" >> $GITHUB_ENV in the docker_build_push job.





# Documentation for Regtech Infrastructure Setup  (Deliverable 5)

## Overview

This RegTech infrastructure is designed for a fictional application that handles sensitive financial data and complies with regulations such as GDPR and PCI-DSS. The pipeline spans code development, security scanning, continuous integration, deployment, monitoring, and traffic management.

## Key Components and Workflow
  ### Developer Environment

A developer pushes code to a GitHub repository.
GitHub Actions are triggered for Continuous Integration (CI).

  ### Code Quality and Security

- SonarQube performs static code analysis to ensure code quality and detect bugs or vulnerabilities.
- Gitleaks scans the code repository for secrets such as API keys or passwords.
- Trivy is used for container image scanning to detect vulnerabilities before pushing the image to AWS Elastic Container Registry (ECR).

  ### CI/CD Pipeline

- After the image scan, the container image is stored in AWS ECR.
- GitHub Actions trigger the deployment by interacting with Argo CD, which manages continuous deployment to Kubernetes clusters.
- Argo CD takes care of deploying the Docker containers to three different Kubernetes environments:
   - Development Cluster
   - Staging Cluster
   - Production Cluster
     
  ### Cluster Setup

The infrastructure comprises three Kubernetes clusters (Prod, Staging, Dev).
Traffic between these clusters is managed using Istio, which provides load balancing, service mesh, and monitoring features.

  ### Monitoring and Logging

- Prometheus is deployed across all clusters for metrics collection and monitoring.
It uses PromQL for querying metrics and integrates with Grafana for data visualization and alerting.
- Loki is used for log aggregation, collecting logs from all clusters and making them accessible for troubleshooting and auditing.
  
## Key Decisions

### Security and Compliance

- Trivy and Gitleaks were chosen for image and secret scanning, ensuring that sensitive information is not leaked in the images and that they are free of vulnerabilities, which is critical for regulatory compliance.
- Using AWS ECR for storing images ensures the security of containerized applications through AWS-managed security policies.

 ### Continuous Deployment with Argo CD

Argo CD was selected for its ability to integrate well with GitOps, allowing for declarative infrastructure management, and enabling continuous deployment across multiple Kubernetes clusters with ease.

 ### Istio for Service Mesh

Istio was chosen for its robust traffic management capabilities, particularly in handling service-to-service communications securely within the Kubernetes clusters. This is essential for a secure, compliant infrastructure where services must communicate with controlled access.

 ### Monitoring with Prometheus and Grafana

Prometheus provides real-time monitoring capabilities, with Grafana used for dashboards and visualization. This setup allows for compliance monitoring and alerting, ensuring that the infrastructure adheres to operational best practices and regulatory requirements.

### Multi-Cluster Kubernetes Deployment

Using separate clusters for Development, Staging, and Production environments ensures the isolation of environments and prevents issues in one environment from affecting others.
Kubernetes clusters are well-suited for scalable, containerized applications and are compliant with many regulatory frameworks due to their fine-grained access controls and resource isolation.

## Use of GitHub Actions for CI/CD

GitHub Actions was chosen for its tight integration with the code repository and its flexibility in defining complex workflows that integrate security and code quality checks into the CI/CD pipeline.

## Deployment and Management Instructions
### Prerequisites
- AWS account with permissions to use ECR, EC2, S3, and IAM.
- Kubernetes clusters deployed on AWS EKS (can be provisioned using Terraform).
- Docker installed locally or on a build server.
- Argo CD installed in the Kubernetes cluster for continuous deployment.
- Prometheus, Grafana, and Loki installed for monitoring and logging.
- Istio service mesh installed across Kubernetes clusters.
  
## Steps for Deployment
  ### Code Commit & Initial CI Setup

- Developers push their code to the GitHub repository.
- GitHub Actions automatically triggers the CI pipeline, running SonarQube for code analysis and Gitleaks for secret scanning.

## Container Build & Security Scanning

- Once the code passes the quality checks, GitHub Actions builds the Docker container.
- The Trivy scanner checks the built container for vulnerabilities.
- The secure image is then pushed to AWS ECR.
  
## Deployment with Argo CD

Argo CD picks up the changes in the code repository and automatically deploys the container to the appropriate Kubernetes environment.
Traffic routing and security between services are managed by Istio.

## Monitoring & Logging

- Prometheus collects metrics from all clusters and Loki collects logs.
- Grafana is used to visualize metrics and set up alerts for infrastructure issues or compliance violations.

## Managing Environments

- Argo CD allows rollback and deployment status tracking for each environment.
- Developers can manage traffic between environments (e.g., promoting from staging to production) through Istio.

## Conclusion
This infrastructure design ensures secure, compliant, and scalable deployments for the RegTech application. Key tools like Argo CD, Istio, Prometheus, and Grafana ensure that the application can be deployed and monitored with compliance to industry regulations while maintaining high availability and performance.
