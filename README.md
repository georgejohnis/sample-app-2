# EKS Web Application

This is a modern web application deployed on Amazon EKS (Elastic Kubernetes Service) using ArgoCD for continuous deployment and AWS Controllers for Kubernetes (ACK) for managing AWS resources.

## Project Structure

```
.
├── frontend/           # React frontend application
├── backend/           # Python FastAPI backend
├── k8s/              # Kubernetes manifests
│   ├── ack/          # AWS Controllers for Kubernetes manifests
│   │   ├── controllers.yaml    # ACK controller installations
│   │   ├── vpc-config.yaml     # VPC, subnets, security groups
│   │   ├── iam-roles.yaml      # IAM roles and policies
│   │   └── ecr-repos.yaml      # ECR repositories
│   ├── argocd-app.yaml  # ArgoCD application configuration
│   ├── kustomization.yaml  # Kustomize configuration
│   └── ...           # Other Kubernetes manifests
└── .github/          # GitHub Actions workflows
```

## Prerequisites

- Node.js >= 18 (for frontend)
- Python >= 3.9
- Docker
- kubectl
- AWS CLI
- eksctl
- pip (Python package manager)
- ArgoCD CLI (optional)

## Getting Started

1. **Create Initial EKS Cluster**:
   ```bash
   # Configure AWS credentials
   aws configure
   
   # Create EKS cluster using eksctl
   eksctl create cluster \
     --name web-app-cluster \
     --region us-west-2 \
     --node-type t3.medium \
     --nodes 2 \
     --nodes-min 2 \
     --nodes-max 3 \
     --managed
   
   # Update kubeconfig
   aws eks update-kubeconfig --name web-app-cluster --region us-west-2
   ```

2. **Install ACK Controllers**:
   ```bash
   # Create namespace for ACK controllers
   kubectl create namespace ack-system
   
   # Install ACK controllers
   kubectl apply -f k8s/ack/controllers.yaml
   ```

3. **Create Additional AWS Resources using ACK**:
   ```bash
   # 1. Create VPC and Network Resources
   kubectl apply -f k8s/ack/vpc-config.yaml
   
   # 2. Create IAM Roles and Policies
   kubectl apply -f k8s/ack/iam-roles.yaml
   
   # 3. Create ECR Repositories
   kubectl apply -f k8s/ack/ecr-repos.yaml
   ```

4. **Wait for Resources to be Ready**:
   ```bash
   # Check VPC status
   kubectl get vpc web-app-vpc
   
   # Check IAM role status
   kubectl get role eks-cluster-role
   
   # Check ECR repositories status
   kubectl get repository frontend-repo backend-repo
   ```

5. **Install ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   
   # Get ArgoCD admin password
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   
   # Port-forward to access ArgoCD UI
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

6. **Apply ArgoCD Application**:
   ```bash
   kubectl apply -f k8s/argocd-app.yaml
   ```

## Development

- Frontend: `cd frontend && npm start`
- Backend: `cd backend && uvicorn main:app --reload`

## CI/CD Pipeline

The application uses a combination of GitHub Actions and ArgoCD for continuous integration and deployment:

1. **GitHub Actions**:
   - Builds Docker images for frontend and backend
   - Pushes images to ECR (created by ACK)
   - Updates Kubernetes manifests with new image tags
   - Commits and pushes changes to the repository

2. **ArgoCD**:
   - Monitors the repository for changes
   - Automatically deploys updates to the EKS cluster
   - Maintains the desired state of the application

## Container Images

The application uses two container images:
- Frontend: React application served by Nginx
- Backend: Python FastAPI application

Both images are built using multi-stage builds to minimize the final image size. The images are stored in Amazon ECR (created by ACK) and pulled by the EKS cluster during deployment.

## License

MIT 