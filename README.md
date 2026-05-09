# 🚀 ArgoCD Helm Image Updater

> A production-grade **CI/CD + GitOps** pipeline on **AWS EKS** using Jenkins, Docker, Helm, Argo CD, and ArgoCD Image Updater — fully automated from code push to deployment.

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS_EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Getting Started](#-getting-started)
- [Pipeline Flow](#-pipeline-flow)
- [Configuration](#-configuration)
- [ArgoCD Image Updater](#-argocd-image-updater-setup)
- [Author](#-author)

---

## 🌟 Overview

This project demonstrates a **fully automated DevOps pipeline** where a single `git push` triggers the entire flow — from building a Docker image to deploying the updated application on a Kubernetes cluster on AWS EKS.

**No manual `kubectl apply`. No SSH into servers. No manual image tag updates.**

### What's automated:
- ✅ Docker image build & push via **Jenkins CI**
- ✅ Automatic image tag detection via **ArgoCD Image Updater**
- ✅ Kubernetes deployment sync via **Argo CD GitOps**
- ✅ Helm-based deployment with rollback support
- ✅ Auto-scaling on **AWS EKS** managed node groups

---

## 🏗 Architecture

```
Developer
    │
    │  git push
    ▼
GitHub Repository
    │
    │  webhook trigger
    ▼
Jenkins CI Pipeline
    │
    ├── Build Docker Image
    ├── Tag with semantic version
    └── Push to Container Registry
                │
                │  new image detected
                ▼
    ArgoCD Image Updater
                │
                │  updates Helm values in Git
                ▼
         Argo CD (GitOps)
                │
                │  syncs cluster state
                ▼
         AWS EKS Cluster
                │
                ├── Helm Chart Deployment
                ├── Auto-scaling Node Groups
                └── Flask Todo Application
```

---

## 🛠 Tech Stack

| Tool | Purpose |
|---|---|
| **AWS EKS** | Managed Kubernetes cluster with auto-scaling |
| **Jenkins** | CI pipeline — build, version, push Docker images |
| **Docker** | Application containerization |
| **Helm** | Kubernetes package manager — deploy, rollback, multi-env |
| **Argo CD** | GitOps continuous delivery — syncs GitHub → EKS |
| **ArgoCD Image Updater** | Automatically detects & updates new container images |
| **GitHub** | Source of truth for both app code and K8s manifests |

---

## 📁 Project Structure

```
argocd-helm-imageupdater/
│
├── flask-app-helm/                  # Helm chart for the Flask application
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       └── service.yaml
│
├── flask-todo-app/                  # Flask application source code
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
│
├── Jenkinsfile                      # Jenkins CI pipeline definition
├── argocd-application.yaml          # Argo CD Application manifest (with image updater annotations)
├── argocd-image-updater-configmap.yaml  # Image updater configuration
├── kind-config.yaml                 # Local cluster config (for development)
└── README.md
```

---

## ✅ Prerequisites

Make sure you have the following installed and configured:

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) — configured with IAM credentials
- [kubectl](https://kubernetes.io/docs/tasks/tools/) — Kubernetes CLI
- [eksctl](https://eksctl.io/) — EKS cluster management
- [Helm](https://helm.sh/docs/intro/install/) v3+
- [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- Jenkins server with Docker installed
- A container registry (DockerHub / AWS ECR)

---

## 🚀 Getting Started

### 1. Create EKS Cluster

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region us-east-1 \
  --version 1.32 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed \
  --zones us-east-1a,us-east-1b
```

### 2. Update kubeconfig

```bash
aws eks update-kubeconfig --name my-eks-cluster --region us-east-1
kubectl get nodes
```

### 3. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 4. Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Get admin password
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### 5. Install ArgoCD Image Updater

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/config/install.yaml
```

### 6. Apply ArgoCD Application

```bash
kubectl apply -f argocd-application.yaml
```

### 7. Access ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
# Open: https://localhost:8080
# Username: admin
```

---

## 🔄 Pipeline Flow

```
1. Developer pushes code to GitHub
         ↓
2. Jenkins detects change via webhook
         ↓
3. Jenkins builds Docker image
   → tags with semantic version (e.g. v1.0.23)
   → pushes to container registry
         ↓
4. ArgoCD Image Updater detects new image tag
   → updates Helm values in Git automatically
         ↓
5. Argo CD detects Git state change
   → syncs Kubernetes cluster
   → deploys new version via Helm
         ↓
6. EKS runs updated application
   → auto-scales based on traffic
```

---

## ⚙️ Configuration

### ArgoCD Application (`argocd-application.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: flask-todo-app
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: myapp=<your-registry>/flask-todo-app
    argocd-image-updater.argoproj.io/myapp.update-strategy: semver
    argocd-image-updater.argoproj.io/write-back-method: git
spec:
  project: default
  source:
    repoURL: https://github.com/Ahad9049/argocd-helm-imageupdater
    path: flask-app-helm
    targetRevision: main
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: flask-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Jenkinsfile (CI Pipeline)

```groovy
pipeline {
  agent any
  environment {
    IMAGE_NAME = "<your-registry>/flask-todo-app"
  }
  stages {
    stage('Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER ./flask-todo-app'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
      }
    }
  }
}
```

---

## 🔧 ArgoCD Image Updater Setup

The Image Updater watches your container registry and automatically updates the image tag in Git when a new image is pushed.

**Key annotations on the ArgoCD Application:**

| Annotation | Description |
|---|---|
| `image-list` | Registry and image to watch |
| `update-strategy: semver` | Updates to latest semantic version |
| `write-back-method: git` | Commits updated tag back to Git |

**Verify Image Updater is running:**

```bash
kubectl get pods -n argocd | grep image-updater
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-image-updater --follow
```

---

## 🗑 Cleanup

```bash
# Delete ArgoCD application
kubectl delete -f argocd-application.yaml

# Delete namespaces
kubectl delete namespace argocd
kubectl delete namespace flask-app

# Delete EKS cluster (avoids AWS charges)
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

---

## 👤 Author

**Abdul Ahad**
- GitHub: [@Ahad9049](https://github.com/Ahad9049)
- Project: [argocd-helm-imageupdater](https://github.com/Ahad9049/argocd-helm-imageupdater)

---

## ⭐ Show your support

If this project helped you, please give it a **star** ⭐ on GitHub!

---

*Built with ❤️ as part of a real-world DevOps learning journey .*
