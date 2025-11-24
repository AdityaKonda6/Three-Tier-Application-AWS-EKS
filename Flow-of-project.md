<img width="840" height="383" alt="image" src="https://github.com/user-attachments/assets/1ec2e7d4-45a3-4fc4-a408-d1274e623dd0" />

---

# **Deployment Architecture Flow Explained**

This diagram represents a modern containerized deployment workflow using **React**, **Node.js**, **MongoDB**, **Docker**, **Amazon ECR**, **Kubernetes**, and a **Load Balancer**.

---

## 🚀 **1. Frontend (React)**

* The React application is containerized using **Docker**.
* The Docker image is pushed to **Amazon ECR (Elastic Container Registry)**.
* Kubernetes later pulls this image to deploy the frontend service.

---

## ⚙️ **2. Backend (Node.js)**

* The Node.js backend is also packaged into a Docker container.
* The image is stored in **Amazon ECR**.
* Kubernetes deploys the backend service using this image.

---

## 🗄️ **3. Database (MongoDB)**

* MongoDB is also containerized.
* Kubernetes runs MongoDB as a StatefulSet or Deployment (depending on configuration).

---

## 📦 **4. Docker Layer**

* Each service (React, Node.js, MongoDB) is first built as a **Docker** image.
* These images act as deployable units for Kubernetes.

---

## 🏛️ **5. Amazon ECR**

* A centralized registry that stores the Docker images.
* Kubernetes clusters pull the images from ECR during deployment.

---

## ☸️ **6. Kubernetes Cluster**

Kubernetes deploys three types of workloads:

* **Frontend Deployment** (React)
* **Backend Deployment** (Node.js)
* **Database Deployment** (MongoDB)

Kubernetes manages:

* Auto scaling
* Self-healing
* Load balancing inside the cluster
* Pod deployments

---

## 🌐 **7. Ingress**

* Acts as an entry point into the Kubernetes cluster.
* Routes external traffic to the correct internal service (frontend or backend).

---

## ⚖️ **8. Load Balancer**

* The Ingress is connected to a **Load Balancer** (e.g., AWS ALB/NLB).
* It distributes incoming traffic to the Kubernetes Ingress controller.
* Ensures high availability and failover.

---

## 👥 **9. Users**

* Users access the application through the Load Balancer endpoint.
* Traffic flows → Load Balancer → Ingress → Kubernetes services → Pods.

---

Project Setup, Prerequisites, and Installation Guide

This guide covers all the prerequisites and initial setup steps required before deploying the three-tier application on AWS EKS.

Prerequisites

Before starting the deployment, ensure you have the following:

AWS Account: With administrative access to create EKS clusters, ECR repositories, and ALB/Route 53 records.

IAM Permissions: Necessary permissions for EKS, ECR, EC2, and the ALB Ingress Controller.

Source Code: Cloned local copies of the backend-chart and frontend-chart repositories/directories.

Installation of Required Tools (From Scratch)

These tools are essential for interacting with AWS and Kubernetes.

1. Install AWS CLI (Command Line Interface)

The AWS CLI is used to configure your environment and interact with AWS services.

# Example for Linux (adjust for your OS)
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
# Verify installation
aws --version


2. Configure AWS Credentials

Configure your AWS Access Key ID and Secret Access Key.

aws configure
# Follow the prompts to enter credentials and region (e.g., ap-south-1)


3. Install kubectl

kubectl is the command-line tool for running commands against Kubernetes clusters.

# Example for Linux
curl -o kubectl [https://s3.us-west-2.amazonaws.com/amazon-eks/1.29/2024-03-01/bin/linux/amd64/kubectl](https://s3.us-west-2.amazonaws.com/amazon-eks/1.29/2024-03-01/bin/linux/amd64/kubectl)
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
# Verify installation
kubectl version --client


4. Install Helm

Helm is the package manager for Kubernetes, used to deploy our applications via charts.

# Example for Linux
curl [https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3](https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3) | bash
# Verify installation
helm version


EKS Cluster Access Configuration

Assuming your EKS cluster is already created, you need to configure kubectl to connect to it.

# Replace <CLUSTER_NAME> and <REGION> with your actual values
aws eks update-kubeconfig --name <CLUSTER_NAME> --region <REGION>

# Verify connection to the cluster
kubectl get nodes


You should see your cluster nodes listed with a Ready status.

Initial Namespace Creation and Docker/ECR Login

1. Create Application Namespace

kubectl create namespace three-tier-app


2. ECR Docker Login (Optional, if required for pulling images)

If your images are in ECR, you need to authenticate Docker.

# Replace <REGION> and <ACCOUNT_ID>
aws ecr get-login-password --region <REGION> | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com


Your environment is now fully prepared for deployment using the commands in the deployment_guide.md.

---

Three-Tier Application Deployment: Command Chronology

This guide documents the exact Helm commands used to deploy, configure, and troubleshoot the Backend and Frontend services, including the final Ingress configuration.

1. Initial Deployment Context

Namespaces: three-tier-app (Backend/Frontend), workshop (MongoDB).

Database Service Name: mongodb-svc.workshop.svc.cluster.local:27017

Release Names: backend-release, frontend-release.

Backend Port: 3500 (API)

Frontend Port: 3000 (UI)

2. Deploying and Troubleshooting the Backend

The backend deployment initially failed due to YAML indentation errors in the livenessProbe configuration.

A. Initial Failed Deployment Command

This command, or similar prior ones, led to the crash:

helm upgrade backend-release ./backend-chart \
  --namespace three-tier-app \
  --set image.repository="[534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend-ecr](https://534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend-ecr)" \
  --set image.tag="latest" \
  --set mongo.serviceName="mongodb-svc.workshop.svc.cluster.local" \
  --set mongo.servicePort=27017


Error: httpGet: field not declared in schema (due to bad indentation in deployment.yaml).

B. YAML Fixes Applied to backend-chart/templates/deployment.yaml

The indentation for the probes was corrected. The paths were also updated (e.g., from /health to /ok and port: http to port: 3500 for a specific application logic).

Original (Incorrect) Snippet:

# INCORRECT SNIPPET
          livenessProbe:
            httpGet:
            path: /health  # Bad indentation


Corrected Snippet:

# CORRECTED SNIPPET
          livenessProbe:
            httpGet:
              path: /ok    # Correct: Nested under httpGet
              port: 3500   # Correct: Nested under httpGet
            initialDelaySeconds: 2 # Correct: Nested under livenessProbe
# ... similar fix for readinessProbe ...


C. Successful Backend Upgrade

The corrected deployment was successfully applied:

helm upgrade backend-release ./backend-chart \
  --namespace three-tier-app \
  --set image.repository="[534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend-ecr](https://534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend-ecr)" \
  --set image.tag="latest" \
  --set mongo.serviceName="mongodb-svc.workshop.svc.cluster.local" \
  --set mongo.servicePort=27017


Verification: kubectl get pods -n three-tier-app showed backend pods in Running status with 1/1 READY.

3. Deploying the Frontend

The frontend was deployed successfully (or assumed to be deployed) with its own chart.

helm upgrade frontend-release ./frontend-chart \
  --namespace three-tier-app \
  --set image.repository="[534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend-ecr](https://534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend-ecr)" \
  --set image.tag="latest" \
  --set service.port=3000


4. Configuring and Deploying the Ingress (External Access)

This step involved adding the Ingress resource via the Frontend chart to provision the AWS ALB for external traffic.

A. YAML Fix Applied to frontend-chart/values.yaml

A syntax error in the ingress.annotations section (yaml: did not find expected ',' or '}') was corrected by switching from JSON-style flow map syntax to standard YAML block style:

Incorrect Snippet:

annotations: {alb.ingress.kubernetes.io/scheme: internet-facing ... }


Corrected Snippet:

annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'


B. Final Helm Upgrade Command (Creating the Ingress)

The final command used helm upgrade to set all necessary Ingress values (ALB annotations, host, and path routing rules) in a single run:

helm upgrade frontend-release ./frontend-chart \
  --namespace three-tier-app \
  --set ingress.enabled=true \
  --set ingress.className="alb" \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/scheme"=internet-facing \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/target-type"=ip \
  --set ingress.annotations."alb\.ingress\.kubernetes\.io/listen-ports"='[{"HTTP": 80}]' \
  --set ingress.hosts[0].host="frontend.amanpathakdevops.study" \
  --set ingress.hosts[0].paths[0].path="/" \
  --set ingress.hosts[0].paths[0].pathType="Prefix" \
  --set ingress.hosts[0].paths[1].path="/api" \
  --set ingress.hosts[0].paths[1].pathType="Prefix" \
  --set ingress.hosts[0].paths[1].backend.service.name="backend-release-backend-chart" \
  --set ingress.hosts[0].paths[1].backend.service.port.number=3500 \
  --set image.repository="[534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend-ecr](https://534232118663.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend-ecr)" \
  --set image.tag="latest" \
  --set service.port=3000


5. Final Verification (External Access)

The application is fully deployed and the ALB is provisioning.

A. Check Load Balancer Status

Run this repeatedly until the ADDRESS field is populated with the AWS DNS name.

kubectl get ingress -n three-tier-app


B. Final Access

Once the ALB address is available, you must set up a CNAME record in your domain's DNS provider to point frontend.amanpathakdevops.study to the ALB address. Then, access the application via:

[http://frontend.amanpathakdevops.study](http://frontend.amanpathakdevops.study)

---

Comprehensive Guide: 3-Tier Application Deployment on AWS EKS

This guide details the end-to-end process of deploying a three-tier application (MongoDB, Node.js API, React Frontend) onto an AWS EKS (Elastic Kubernetes Service) cluster, including environment setup, containerization, Kubernetes manifests, and external traffic routing via the AWS Application Load Balancer (ALB) Ingress Controller.

1. Project Overview and Architecture

The application follows a standard three-tier architecture:

Tier

Component

Technology

Role

Data (Tier 3)

MongoDB

mongo:4.4.6 Docker Image

Persistent storage for application data.

Application (Tier 2)

Backend API

Node.js (Example)

Business logic, connects to MongoDB. Runs on port 3500.

Presentation (Tier 1)

Frontend UI

React.js (Example)

User interface. Runs on port 3000.

External Access

Ingress

AWS ALB Controller

Routes external traffic based on host and path.

2. Phase 1: Environment Setup and Tool Installation

This phase covers the necessary command-line tools and initial AWS configuration.

A. AWS EC2 Instance Setup (Prerequisite)

Launch an EC2 instance (e.g., Ubuntu AMI) to act as your control machine.

Configure a Security Group to allow SSH access.

B. Docker Installation and Setup

Install Docker on the control machine to build and push container images.

#!/bin/bash
# Update the apt package index
sudo apt update
# Install necessary dependencies
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
# Add Docker's official GPG key
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo apt-key add -
# Add the Docker repository (Example for Ubuntu Focal/Jammy)
sudo add-apt-repository "deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) focal stable"
# Update and install Docker CE
sudo apt update
sudo apt install -y docker-ce
# Add current user to the docker group
sudo usermod -aG docker $USER
echo "Docker installed. You may need to log out and log back in."

---

# Use the official Node.js 14 image as the base image
FROM node:14
# Set the working directory inside the container
WORKDIR /usr/src/app
# Copy package files and install dependencies
COPY package*.json ./
RUN npm install
# Copy the rest of the application code
COPY . .
# Specify the command to run when the container starts
CMD [ "npm", "start" ]
```

**Build and Tag Images:**
```bash
# Build the frontend image
docker build -t frontend .
# Build the backend image
docker build -t backend .
```

### D. AWS CLI Installation and ECR Push

Install AWS CLI and configure credentials to push images to Amazon ECR.

```bash
# Install AWS CLI v2
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update

# Configure AWS credentials
aws configure

# Push to ECR (Replace ACCOUNT_ID and REGION)
aws ecr get-login-password --region <REGION> | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
# Tag and push your images
docker tag frontend <ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/frontend:latest](https://.amazonaws.com/frontend:latest)
docker push <ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/frontend:latest](https://.amazonaws.com/frontend:latest)
```

## 3. Phase 2: EKS Cluster and Kubernetes Tools

Install the necessary tools to manage the EKS cluster.

### A. Install kubectl and eksctl

```bash
# Install kubectl
curl -o kubectl [https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl](https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl)
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

# Install eksctl
curl --silent --location "[https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname](https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname) -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### B. Setup EKS Cluster

Use `eksctl` to create the cluster and configure your local kubeconfig.

```bash
# Create the EKS cluster (this takes time)
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2

# Update kubeconfig to connect kubectl to the new cluster
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster

# Verify nodes are ready
kubectl get nodes
```
---

# --- 1. Kubernetes Secret (mongo-sec) ---
apiVersion: v1
kind: Secret
metadata:
  namespace: three-tier
  name: mongo-sec
type: Opaque
data:
  password: cGFzc3dvcmQxMjM= # Base64 encoded: "password123"
  username: YWRtaW4= # Base64 encoded: "admin"
---
# --- 2. Persistent Volume (mongo-pv) using HostPath (for demonstration) ---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
  namespace: three-tier
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  hostPath:
    # IMPORTANT: hostPath is for testing only. Use EBS/EFS in production.
    path: /data/db
---
# --- 3. Persistent Volume Claim (mongo-volume-claim) ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-volume-claim
  namespace: three-tier
spec:
  accessModes:
  - ReadWriteOnce
  # Note: storageClassName is typically set to a provisioner like "gp2" in EKS, but left empty here for hostPath binding.
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
---
# --- 4. MongoDB Service (mongodb-svc) ---
apiVersion: v1
kind: Service
metadata:
  namespace: three-tier
  name: mongodb-svc
spec:
  selector:
    app: mongodb
  ports:
  - name: mongodb-svc
    protocol: TCP
    port: 27017
    targetPort: 27017
---
# --- 5. MongoDB Deployment (mongodb) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: three-tier
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mon
        image: mongo:4.4.6
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-sec
              key: username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-sec
              key: password
        volumeMounts:
        - name: mongo-volume
          mountPath: /data/db
      volumes:
      - name: mongo-volume
        persistentVolumeClaim:
          claimName: mongo-volume-claim
```
### B. Apply MongoDB Manifests

```bash
# Create the namespace first (if not already done)
kubectl create namespace three-tier

# Apply the single manifest file
kubectl apply -f mongo_manifests.yaml

# Verify deployment
kubectl get pods -n three-tier -l app=mongodb
```
---

# --- 1. Backend Service (api) ---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: three-tier
spec:
  ports:
  - port: 3500
    protocol: TCP
    type: ClusterIP
  selector:
    role: api
---
# --- 2. Backend Deployment (api) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: three-tier
  labels:
    role: api
    env: demo
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 25%
  selector:
    matchLabels:
      role: api
  template:
    metadata:
      labels:
        role: api
    spec:
      # Requires a secret for ECR login, named ecr-registry-secret
      imagePullSecrets:
      - name: ecr-registry-secret
      containers:
      - name: api
        image: <YOUR_ECR_ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/backend:latest](https://.amazonaws.com/backend:latest)
        imagePullPolicy: Always
        env:
        - name: MONGO_CONN_STR
          # Connection string uses the internal service name: mongodb://<Service Name>:<Port>/<Database>
          value: mongodb://mongodb-svc:27017/todo?directConnection=true
        - name: MONGO_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-sec
              key: username
        - name: MONGO_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-sec
              key: password
        ports:
        - containerPort: 3500
        livenessProbe:
          httpGet:
            path: /ok
            port: 3500
          initialDelaySeconds: 2
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ok
            port: 3500
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
---

# --- 1. Frontend Service (frontend) ---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: three-tier
spec:
  ports:
  - port: 3000
    protocol: TCP
    type: ClusterIP
  selector:
    role: frontend
---
# --- 2. Frontend Deployment (frontend) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: three-tier
  labels:
    role: frontend
    env: demo
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 25%
  selector:
    matchLabels:
      role: frontend
  template:
    metadata:
      labels:
        role: frontend
    spec:
      # Requires a secret for ECR login, named ecr-registry-secret
      imagePullSecrets:
      - name: ecr-registry-secret
      containers:
      - name: frontend
        image: <YOUR_ECR_ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/frontend:latest](https://.amazonaws.com/frontend:latest)
        imagePullPolicy: Always
        env:
        # Frontend needs to know the external URL for the API
        - name: REACT_APP_BACKEND_URL
          value: "[http://backend.amanpathakdevops.study/api](http://backend.amanpathakdevops.study/api)"
        ports:
        - containerPort: 3000
```

### C. Apply Backend and Frontend Manifests

```bash
kubectl apply -f backend_manifests.yaml
kubectl apply -f frontend_manifests.yaml

# Verify all application pods are running
kubectl get pods -n three-tier
```

## 6. Phase 5: External Access via ALB Ingress Controller

To expose the application to the internet, we use the AWS Load Balancer Controller to manage an ALB based on an Ingress resource.

### A. Install AWS Load Balancer Controller

This involves creating an IAM policy, associating the OIDC provider, and deploying the controller using Helm.

```bash
#!/bin/bash
# 1. Download IAM Policy Document
curl -O [https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json)

# 2. Create IAM Policy (Replace 123456789012 with your AWS Account ID if needed)
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# 3. Associate OIDC Provider with EKS Cluster (Replace placeholders)
eksctl utils associate-iam-oidc-provider --region=<REGION> --cluster=<CLUSTER_NAME> --approve

# 4. Create IAM Service Account for the Controller (Replace placeholders)
eksctl create iamserviceaccount --cluster=<CLUSTER_NAME> --namespace=kube-system --name=aws-load-balancer-controller \
    --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=<REGION>

# 5. Install AWS Load Balancer Controller using Helm
helm repo add eks [https://aws.github.io/eks-charts](https://aws.github.io/eks-charts)
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=<CLUSTER_NAME> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
```
---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mainlb
  namespace: three-tier
  annotations:
    # Use internet-facing scheme for public access
    alb.ingress.kubernetes.io/scheme: internet-facing
    # Use IP target type for ALB to communicate directly with Pods
    alb.ingress.kubernetes.io/target-type: ip
    # Define listeners for HTTP on port 80
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
  # The article uses a single host for routing everything, which is non-standard but works for this setup
  - host: backend.amanpathakdevops.study
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 3500
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 3000
```

### C. Apply Ingress and Get Endpoint

```bash
# Apply the Ingress manifest
kubectl apply -f ingress.yaml

# Monitor the Ingress until the ALB address appears
kubectl get ingress -n three-tier
```
---

To avoid recurring AWS charges, delete the EKS cluster when the project is finished.

```bash
eksctl delete cluster --name three-tier-cluster --region us-west-2
```
