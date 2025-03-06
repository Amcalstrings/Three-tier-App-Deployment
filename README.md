# 3-Tier Application Deployment on Kubernetes

This repository contains Kubernetes manifests and deployment instructions for a **3-tier application** consisting of a **frontend, backend, and MongoDB database**. The deployment includes **CI/CD integration using GitHub Actions**, **Application Load Balancer (ALB) setup**, and **Ingress configuration**.

## Project Overview

- **Frontend**: React application
- **Backend**: Node.js API
- **Database**: MongoDB
- **CI/CD**: GitHub Actions for automated deployment
- **Cloud Provider**: AWS (EC2, IAM, ALB, EKS, etc.)
- **Orchestration**: Kubernetes
- **Containerization**: Docker

---

## 📌 Phase 1: Setting Up EC2, IAM User, and Basic Tools

### 1️⃣ Launch an EC2 Instance
- Create an EC2 instance (Amazon Linux 2 or Ubuntu 20.04 recommended)
- Open required ports: **22, 80, 443** (Ingress rules in Security Group)
- SSH into the EC2 instance:
  ```sh
  ssh -i your-key.pem ubuntu@your-ec2-public-ip
  ```

### 2️⃣ Install Required Tools
```sh
sudo apt update && sudo apt install -y docker.io kubectl awscli
```

### 3️⃣ Configure IAM User for Kubernetes
- Create an IAM user with the following permissions:
  - `AmazonEC2FullAccess`
  - `AmazonEKSClusterPolicy`
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEKSServicePolicy`
  - `AdministratorAccess`
- Configure AWS CLI:
  ```sh
  aws configure
  ```

---

## 📌 Phase 2: Build and Push Docker Images

### 1️⃣ Clone the Repository
```sh
git clone <your-repo-url>
cd <your-repo>
```

### 2️⃣ Build and Push Docker Images
```sh
# Build backend image
cd backend
docker build -t <your-dockerhub-username>/backend:latest .
docker push <your-dockerhub-username>/backend:latest

# Build frontend image
cd ../frontend
docker build -t <your-dockerhub-username>/frontend:latest .
docker push <your-dockerhub-username>/frontend:latest
```

🔹 **Update Kubernetes manifests** with the correct image names before deployment.

---

## 📌 Phase 3: Deploying Kubernetes Manifests

### 1️⃣ Set Up Kubernetes Cluster
- Create an **EKS cluster** (if not already done):
  ```sh
  eksctl create cluster --name three-tier-app --region us-east-1
  ```
- Verify Cluster is Running:
  ```sh
  kubectl get nodes
  ```

### 2️⃣ Deploy Application to Kubernetes
```sh
kubectl apply -f k8s-manifests/
```

### 3️⃣ Verify Deployment
```sh
kubectl get pods -n workshop
kubectl get svc -n workshop
kubectl get deployments -n workshop
```

---

## 📌 Phase 4: Configure ALB & Ingress

### 1️⃣ Install AWS Load Balancer Controller
```sh
kubectl apply -f https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.2.0/v2_2_0_full.yaml
```

### 2️⃣ Apply Ingress Manifests
```sh
kubectl apply -f k8s-manifests/ingress.yaml
```

### 3️⃣ Get Load Balancer URL
```sh
kubectl get ingress -n workshop
```
🔹 Copy and paste the ALB URL in your browser to access the application.

---

## 📌 Phase 5: Destroy Everything

### 1️⃣ Delete Kubernetes Resources
```sh
kubectl delete -f k8s-manifests/
```

### 2️⃣ Delete EKS Cluster
```sh
eksctl delete cluster --name three-tier-app
```

### 3️⃣ Delete Docker Images (Optional)
```sh
docker rmi <your-dockerhub-username>/backend:latest

docker rmi <your-dockerhub-username>/frontend:latest
```

---

## 🛠️ CI/CD Pipeline (GitHub Actions)

1️⃣ Create a **`.github/workflows/deploy.yml`** file in your repository.

2️⃣ Add the following workflow:
```yaml
name: Deploy to Kubernetes
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2
      - name: Setup Kubernetes
        uses: azure/setup-kubectl@v1
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s-manifests/
```

3️⃣ **Push your code to GitHub**, and the workflow will automatically deploy your application.

---

## 🎯 Conclusion

This guide outlines the **end-to-end deployment** of a **3-tier application on Kubernetes** using **AWS, Docker, Kubernetes, and GitHub Actions CI/CD**.

🚀 Happy Deploying! 🚀

