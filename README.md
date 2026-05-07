# 🚀 ArgoCD Helm Image Updater

> A fully automated GitOps CI/CD pipeline — push a Docker image and watch Kubernetes deploy it automatically. No manual steps. No human error.

---

## 📌 Overview

This project implements a **production-grade GitOps pipeline** using ArgoCD and the ArgoCD Image Updater. When a new Docker image is pushed to Docker Hub, the Image Updater automatically detects the change, commits the updated image tag to this GitHub repository, and ArgoCD syncs the deployment to Kubernetes — all without any manual intervention.

---

## ⚙️ How It Works

```
Docker Hub → Argo CD Image Updater → GitHub (auto-commit) → Argo CD Sync → Kubernetes
```

1. You push a new Docker image to **Docker Hub**
2. **Argo CD Image Updater** detects the new image version
3. It automatically **commits** the updated tag to this repo
4. **Argo CD** detects the Git change and **syncs** to Kubernetes
5. Your app is **live** — zero manual steps 🎉

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Kubernetes (Kind)** | Local cluster for running workloads |
| **Helm** | Kubernetes package manager for templating |
| **Argo CD** | GitOps continuous delivery tool |
| **Argo CD Image Updater** | Watches Docker Hub & auto-updates image tags |
| **Flask** | Sample application being deployed |
| **Docker Hub** | Container image registry |

---

## 📁 Project Structure

```
argocd-helm-imageupdater/
│
├── flask-app-helm/                  # Helm chart for Flask application
│   ├── templates/
│   ├── Chart.yaml
│   └── values.yaml
│
├── argocd-application.yaml          # ArgoCD Application manifest
├── argocd-image-updater-configmap.yaml  # Image Updater configuration
├── kind-config.yaml                 # Kind cluster configuration
├── nohup.out                        # ArgoCD Image Updater logs
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

Make sure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

---

### 1️⃣ Create a Kind Cluster

```bash
kind create cluster --config kind-config.yaml
```

### 2️⃣ Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3️⃣ Install Argo CD Image Updater

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

### 4️⃣ Apply Image Updater Config

```bash
kubectl apply -f argocd-image-updater-configmap.yaml
```

### 5️⃣ Deploy the Application

```bash
kubectl apply -f argocd-application.yaml
```

### 6️⃣ Access Argo CD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open [https://localhost:8080](https://localhost:8080) in your browser.

---

## 🔄 Automated Image Update Flow

Once everything is set up, the automation kicks in:

```bash
# Simply push a new image to Docker Hub
docker build -t yourusername/flask-app:1.0.1 .
docker push yourusername/flask-app:1.0.1

# Argo CD Image Updater does the rest ✅
```

---

## 💡 Key Features

- ✅ **Zero-touch deployments** — fully automated end to end
- ✅ **GitOps principle** — Git is the single source of truth
- ✅ **Self-healing** — Argo CD auto-corrects any drift
- ✅ **Auditable** — every change is tracked in Git history
- ✅ **Version-controlled** — easy rollback to any previous state
- ✅ **Enterprise-grade** — the same pattern used by Fortune 500 companies

---

## 📸 Architecture Diagram

```
┌─────────────┐     push      ┌─────────────┐
│  Developer  │ ────────────► │  Docker Hub │
└─────────────┘               └──────┬──────┘
                                     │ detect new image
                              ┌──────▼──────────────┐
                              │ ArgoCD Image Updater │
                              └──────┬───────────────┘
                                     │ auto-commit tag
                              ┌──────▼──────┐
                              │   GitHub    │
                              └──────┬──────┘
                                     │ sync
                              ┌──────▼──────┐
                              │   Argo CD   │
                              └──────┬──────┘
                                     │ deploy
                              ┌──────▼──────┐
                              │ Kubernetes  │
                              └─────────────┘
```

---

## 👨‍💻 Author

**Abdul Ahad**
- GitHub: [@Ahad9049](https://github.com/Ahad9049)

---

## ⭐ Show Your Support

If this project helped you, please consider giving it a **star** ⭐ — it means a lot and helps others find it!

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
