# Argo CD Learning Repository & Command Reference

This repository contains Helm charts, Argo CD Application manifests, and AppProject configurations used to deploy applications on a local Kubernetes cluster (Minikube / KIND / Docker Desktop).

---

## 📋 Table of Contents
1. [Installing Argo CD via Helm](#1-installing-argo-cd-via-helm)
2. [CLI Configuration & Authentication](#2-cli-configuration--authentication)
3. [Managing Argo CD Passwords](#3-managing-argo-cd-passwords)
4. [Helm Chart Commands](#4-helm-chart-commands)
5. [Git & GitHub Commands](#5-git--github-commands)
6. [Deploying Argo CD Applications](#6-deploying-argo-cd-applications)
7. [Creating and Managing AppProjects](#7-creating-and-managing-appprojects)
8. [Troubleshooting & Handy Commands](#8-troubleshooting--handy-commands)

---

## 1. Installing Argo CD via Helm

### Add Argo Helm Repository
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### Install Argo CD into `argo-cd` Namespace
```bash
# Create namespace
kubectl create namespace argo-cd

# Install Argo CD
helm install argo-cd charts/argo-cd/ --namespace argo-cd --create-namespace
```

> **Note on CRD ownership conflicts:** If Helm complains about existing CRDs, delete leftover CRDs or use `--take-ownership`:
> ```bash
> kubectl delete crd applications.argoproj.io applicationsets.argoproj.io appprojects.argoproj.io argocdextensions.argoproj.io
> ```

---

## 2. CLI Configuration & Authentication

### Set default `kubectl` namespace to `argo-cd`
```bash
kubectl config set-context --current --namespace=argo-cd
```

### Get Initial Admin Password
```bash
kubectl get secret argocd-initial-admin-secret -n argo-cd -o jsonpath="{.data.password}" | base64 -d; echo
```

### Configure Shell Options for Argo CD CLI
Set `ARGOCD_OPTS` to enable built-in port-forwarding, bypass TLS cert checks, and use gRPC-web (prevents `kubectl port-forward` connection resets):
```bash
export ARGOCD_OPTS="--port-forward --port-forward-namespace argo-cd --insecure --grpc-web"
```
*(Add this export to your `~/.zshrc` or `~/.bashrc` to make it permanent).*

### Log In to Argo CD CLI
```bash
argocd login --username admin --password <YOUR_PASSWORD> --name local
```

---

## 3. Managing Argo CD Passwords

### Update Password via CLI
> **Requirement:** Password length must be between **8 and 32 characters** (`^.{8,32}$`).

```bash
# Interactive mode
argocd account update-password

# Non-interactive mode
argocd account update-password \
  --current-password "<CURRENT_PASSWORD>" \
  --new-password "<NEW_PASSWORD>"
```

---

## 4. Helm Chart Commands

### Test & Lint Local Helm Chart (`my-helm-app/`)
```bash
# Lint chart syntax
helm lint my-helm-app/

# Render templates locally to inspect generated YAML
helm template my-helm-app/
```

---

## 5. Git & GitHub Commands

### Initialize and Push Repository
```bash
# Initialize local repo and rename branch to main
git init
git branch -M main

# Add files and commit
git add .
git commit -m "Initial Argo CD learning setup"

# Link to GitHub repository
git remote add origin https://github.com/irajatpandey/argo-cd-learning.git

# Push to GitHub
git push -u origin main
```

---

## 6. Deploying Argo CD Applications

### Deploying a Single Application (`kind: Application`)
File: `argo-configs/deploy-second-application/ application-2.yaml`
```bash
kubectl apply -f "argo-configs/deploy-second-application/ application-2.yaml"
```

### Deploying Multi-Environment AppSet (`kind: ApplicationSet`)
File: `argo-configs/deploy-first-application/helm-applicationset.yaml`
```bash
kubectl apply -f argo-configs/deploy-first-application/helm-applicationset.yaml
```

### Syncing & Checking Application Status
```bash
# Check application status
argocd app get todo-app

# Sync application manually
argocd app sync todo-app

# Force sync application
argocd app sync todo-app --force
```

---

## 7. Creating and Managing AppProjects

An `AppProject` (`kind: AppProject`) provides RBAC boundaries and restricts target namespaces, repos, and resource types.

File: `argo-configs/argo-project/my-project.yaml`
```bash
kubectl apply -f argo-configs/argo-project/my-project.yaml
```

### List and Inspect AppProjects
```bash
# List project CRDs via kubectl
kubectl get appproject -n argo-cd

# Inspect AppProject details
kubectl get appproject my-project -n argo-cd -o yaml
```

---

## 8. Troubleshooting & Handy Commands

### View Cluster Pods and Events
```bash
# Get pods in default namespace
kubectl get pods -n default

# Get pods in argo-cd namespace
kubectl get pods -n argo-cd

# View events sorted by timestamp
kubectl get events -n default --sort-by='.metadata.creationTimestamp'

# Describe pod for detailed status/events
kubectl describe pod <POD_NAME> -n default
```

### Terminate Stuck Argo CD Operations
If Argo CD gets stuck in `another operation is already in progress`:
```bash
argocd app terminate-op todo-app
```
