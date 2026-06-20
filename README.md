# Employee Platform

Self-service employee onboarding on AWS EKS. HR provisions containerized Ubuntu workspaces in under 2 minutes.

## How it works

HR staff log into a web portal and create a new employee by filling in a name, email, department, and role. This triggers a backend API that creates a Kubernetes deployment with a department-specific container image, registers a personal DNS record in Route53, and stores the employee record in DynamoDB.

Employees connect via OpenVPN and access their workspace through a browser at their personal URL. Three workspace types are available: Developer (VS Code, Git, Node.js), HR (LibreOffice, browsers), and Infrastructure (Docker, kubectl, Terraform, AWS CLI). AWS Cognito handles authentication and automatic deprovisioning when employees leave.

The cluster spans 3 availability zones in eu-west-1 on EKS with 3 t3.medium worker nodes. Grafana and Prometheus provide real-time monitoring of workspace status and resource usage.

## Screenshots

![HR Portal](images/workspace-employees.png)

![Infrastructure Workspace](images/marcuswright.png)

![Monitoring Overview](images/monitoring-1.png)

![Monitoring Details](images/monitoring-2.png)

## Tech stack

| Component | Technology |
|-----------|------------|
| Orchestration | Amazon EKS (Kubernetes v1.27) |
| Infrastructure | Terraform |
| Frontend | React + nginx |
| Backend | Node.js API |
| Database | DynamoDB |
| Identity | AWS Cognito |
| DNS | Route53 private hosted zone |
| Monitoring | Prometheus + Grafana + Loki |
| VPN | OpenVPN |

## Setup

```bash
cd terraform
terraform init
terraform apply -auto-approve
```

```bash
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <account>.dkr.ecr.eu-west-1.amazonaws.com
cd applications/workspaces
docker build -t workspace-dev:latest -f dev/Dockerfile .
docker push <account>.dkr.ecr.eu-west-1.amazonaws.com/workspace-dev:latest
aws eks update-kubeconfig --name innovatech-eks --region eu-west-1
kubectl apply -f kubernetes/
```

Deployment takes approximately 20 minutes.
