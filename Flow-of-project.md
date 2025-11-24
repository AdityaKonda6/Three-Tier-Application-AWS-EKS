# 🌐 Three-Tier Application Deployment on **AWS EKS**

A comprehensive guide to deploying a **React Frontend**, **Node.js Backend**, and **MongoDB** database on **Amazon EKS** using **Docker**, **ECR**, **Helm**, **Kubernetes**, and the **AWS Load Balancer Controller**.

---

# 🏗️ Deployment Architecture

<img width="840" height="383" alt="image" src="https://github.com/user-attachments/assets/1ec2e7d4-45a3-4fc4-a408-d1274e623dd0" />


---

# 📘 Table of Contents

* [Overview](#overview)
* [Architecture Flow](#architecture-flow)
* [Prerequisites](#prerequisites)
* [Tool Installation](#tool-installation)
* [Cluster Access Setup](#cluster-access-setup)
* [Deployment Chronology](#deployment-chronology)
* [MongoDB Deployment](#mongodb-deployment)
* [Backend Deployment](#backend-deployment)
* [Frontend Deployment](#frontend-deployment)
* [Ingress / External Access](#ingress--external-access)
* [Verification](#verification)
* [Cleanup](#cleanup)

---

# 🧩 Overview

This project deploys a **three-tier cloud-native application** on **AWS EKS**:

| Tier                | Component                  | Technology                     | Description                             |
| ------------------- | -------------------------- | ------------------------------ | --------------------------------------- |
| 1️⃣ Presentation    | React Frontend             | Docker, Kubernetes             | Provides UI, runs on port **3000**      |
| 2️⃣ Application     | Node.js Backend API        | Docker, Kubernetes             | Handles business logic on port **3500** |
| 3️⃣ Data            | MongoDB                    | Stateful Kubernetes Deployment | Stores application data                 |
| 🌐 External Routing | AWS ALB Ingress Controller | ALB                            | Publishes services publicly             |

---

# 🔁 Architecture Flow

## 1. **Frontend (React)**

* Dockerized and pushed to **Amazon ECR**
* Deployed using **Helm** in Kubernetes

## 2. **Backend (Node.js)**

* Packaged as Docker image → stored in ECR
* Deployed via Helm with environment variables and MongoDB connection string

## 3. **MongoDB**

* Runs as a Kubernetes Deployment + Persistent Volume Claim
* Secured with k8s Secrets

## 4. **Docker**

* All components are built and tagged locally
* Images pushed to ECR for deployment

## 5. **ECR (Elastic Container Registry)**

* Stores Docker images consumed by Kubernetes

## 6. **Kubernetes Cluster**

Handles:

* Rolling updates
* Auto-healing
* Networking
* Scaling
* Pod lifecycle

## 7. **Ingress**

* AWS ALB Ingress routes external traffic
* Separate routes for frontend (`/`) and backend (`/api`)

## 8. **Load Balancer**

* Public ALB endpoint
* Routes traffic to Ingress Controller

## 9. **Users**

* Access the application via ALB FQDN or domain name

---

# 📦 Prerequisites

Before deploying, ensure you have:

* An **AWS account**
* IAM permissions for:

  * EKS
  * ECR
  * IAM Roles
  * ALB Ingress Controller
* Local copies of:

  * `backend-chart/`
  * `frontend-chart/`

---

# 🛠️ Tool Installation

### 1. Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### 2. Configure AWS Credentials

```bash
aws configure
```

### 3. Install kubectl

```bash
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.29/latest/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### 4. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version
```

---

# ☸️ Cluster Access Setup

```bash
aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>
kubectl get nodes
```

---

# 🚀 Deployment Chronology

Below is the ~exact sequence~ used to deploy your application.

---

# 🍃 MongoDB Deployment

### Apply your MongoDB manifest:

```bash
kubectl apply -f mongo_manifests.yaml
kubectl get pods -n three-tier
```

This includes:

* Secret
* PersistentVolume
* PersistentVolumeClaim
* Service
* Deployment

---

# ⚙️ Backend Deployment

### Fix applied: liveness/readiness probe indentation and path update (`/ok`)

### Deploy with Helm:

```bash
helm upgrade backend-release ./backend-chart \
  --namespace three-tier-app \
  --set image.repository="534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend-ecr" \
  --set image.tag="latest" \
  --set mongo.serviceName="mongodb-svc.workshop.svc.cluster.local" \
  --set mongo.servicePort=27017
```

### Validate

```bash
kubectl get pods -n three-tier-app
```

---

# 🎨 Frontend Deployment

```bash
helm upgrade frontend-release ./frontend-chart \
  --namespace three-tier-app \
  --set image.repository="534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend-ecr" \
  --set image.tag="latest" \
  --set service.port=3000
```

---

# 🌍 Ingress / External Access

### Fix applied:

Converted JSON-style annotations to YAML block style.

### Apply Ingress using Helm:

```bash
helm upgrade frontend-release ./frontend-chart \
  --namespace three-tier-app \
  --set ingress.enabled=true \
  --set ingress.className="alb" \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/scheme"=internet-facing \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/target-type"=ip \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/listen-ports"='[{"HTTP":80}]' \
  --set ingress.hosts[0].host="frontend.amanpathakdevops.study" \
  --set ingress.hosts[0].paths[0].path="/" \
  --set ingress.hosts[0].paths[0].pathType="Prefix" \
  --set ingress.hosts[0].paths[1].path="/api" \
  --set ingress.hosts[0].paths[1].pathType="Prefix" \
  --set ingress.hosts[0].paths[1].backend.service.name="backend-release-backend-chart" \
  --set ingress.hosts[0].paths[1].backend.service.port.number=3500
```

---

# 🔍 Verification

### Check ALB hostname:

```bash
kubectl get ingress -n three-tier-app
```

When ALB provisioning is complete, add a **CNAME record** pointing:

```
frontend.amanpathakdevops.study --> <ALB DNS Name>
```

Then access your app:

```
http://frontend.amanpathakdevops.study
```

---

# 🧹 Cleanup

```bash
eksctl delete cluster --name three-tier-cluster --region us-west-2
```


