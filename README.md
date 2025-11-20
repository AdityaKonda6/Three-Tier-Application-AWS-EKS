# Kubernetes & kubectl Comprehensive Cheat Sheet

This document consolidates all key commands and information extracted from the provided cheat sheets, and also includes a full guide on Amazon EKS (Elastic Kubernetes Service).

---

# 🧭 **kubectl Cheat Sheet**

## ## **Listing Resources**

* `kubectl get namespaces` – List all namespaces
* `kubectl get pods` – List all pods in current namespace
* `kubectl get pods -o wide` – Detailed pod list
* `kubectl get pods --field-selector=<selector>,nodeName=<server-name>` – List pods on specific node
* `kubectl get replicationcontroller [name]` – List replication controllers
* `kubectl get replicationcontroller,services` – List replication controllers & services
* `kubectl get daemonset` – List daemon sets

---

## ## **Displaying State of Resources**

* `kubectl describe nodes [name]`
* `kubectl describe pods [name]`
* `kubectl describe -f pod.json`
* `kubectl describe pods [replication-controller-name]`
* `kubectl describe pods`

---

## ## **Printing Container Logs**

* `kubectl logs [pod-name]` – Print logs
* `kubectl logs -f [pod-name]` – Stream logs

---

## ## **Deleting Resources**

* `kubectl delete -f pod.yaml`
* `kubectl delete pods,services -l [key]=[value]`
* `kubectl delete pods --all`

---

## ## **Creating Resources**

* `kubectl create namespace <namespace>`
* `kubectl create -f <file>` – Create resources from YAML/JSON

---

## ## **Applying & Updating Resources**

* `kubectl apply -f <service>.yaml`
* `kubectl apply -f <controller>.yaml`
* `kubectl apply -f <directory>`
* `kubectl edit svc/<service-name>`

---

## ## **Executing Commands in Pods**

* `kubectl exec [pod] -- [command]`
* `kubectl exec [pod] -c [container] -- [command]`
* `kubectl exec -it [pod] -- /bin/bash`

---

## ## **Modifying kubeconfig**

* `kubectl config current-context`
* `kubectl config get-contexts`
* `kubectl config use-context <context>`
* `kubectl config set-context --cluster=<cluster> --user=<user>`
* `kubectl config unset <property>`

---

# 📦 **Resource Types – Short Names**

| Short  | Full Name                  |
| ------ | -------------------------- |
| csr    | certificatesigningrequests |
| cs     | componentstatuses          |
| cm     | configmaps                 |
| ds     | daemonsets                 |
| deploy | deployments                |
| ep     | endpoints                  |
| ev     | events                     |
| hpa    | horizontalpodautoscalers   |
| ing    | ingresses                  |
| limits | limitranges                |
| ns     | namespaces                 |
| no     | nodes                      |
| pvc    | persistentvolumeclaims     |
| pv     | persistentvolumes          |
| po     | pods                       |
| pdb    | poddisruptionbudgets       |
| psp    | podsecuritypolicies        |
| rs     | replicasets                |
| rc     | replicationcontrollers     |
| quota  | resourcequotas             |
| sa     | serviceaccounts            |
| svc    | services                   |

---

# 🛠 **Additional kubectl Commands (from second sheet)**

## ## **Global Flags**

* `--namespace <name>` – Specify namespace
* `--help` – Show help

## ## **Context & Configuration**

* `kubectl config get-contexts`
* `kubectl config current-context`
* `kubectl config use-context <context>`
* `kubectl config delete-context <context>`

---

## ## **Display Resources**

* `kubectl get <resource>` – list all
* `kubectl get <resource> -o wide`
* `kubectl get <resource> -A` – all namespaces
* `kubectl get <resource> <name>` – single resource
* `kubectl get <resource> <name> -o yaml`
* `kubectl get <resource> -l <key>=<value>`
* `kubectl describe <resource>`

---

## ## **Create Resources Manually**

* `kubectl run <name> --image=<image>` – Run pod
* `kubectl create deployment <name> --image=<image>`
* `kubectl expose pod <pod> --port=<port>` – Service
* `kubectl expose deployment <name> --port=<port>`
* `kubectl create ingress <name> --rule=<host/path=svc:port>`
* `kubectl create job <name> --image=<image>`
* `kubectl create job <name> --from=cronjob/<name>`
* `kubectl create cronjob <name> --image=<image> --schedule=<cron>`
* `kubectl create secret generic <name> --from-literal=<key>=<value>`
* `kubectl create secret docker-registry <name> --docker-server=<server> --docker-username=<user> --docker-password=<password>`

---

## ## **Generate YAML Manifests**

* `kubectl create deployment <name> --image=<image> --dry-run=client -o yaml`
* `kubectl expose deployment <name> --port=<port> --dry-run=client -o yaml`

---

# ☸ **Amazon EKS (Elastic Kubernetes Service)**

## ## **What is EKS?**

Amazon EKS is a managed Kubernetes service by AWS that automates:

* Control plane provisioning
* Node management (via Node Groups or Fargate)
* Scaling & security updates
* Cluster integrations with IAM, VPC, ALB, CloudWatch, etc.

---

# 🚀 **Steps to Create an EKS Cluster**

## ## **1. Install Required Tools**

```
aws CLI
kubectl
eksctl
```

---

## ## **2. Configure AWS CLI**

```
aws configure
```

Provide Access Key, Secret Key, Region, Output format.

---

## ## **3. Create EKS Cluster Using eksctl**

```
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name ng1 \
  --node-type t3.medium \
  --nodes 3
```

---

## ## **4. Verify Cluster**

```
kubectl get nodes
kubectl get pods -A
```

---

## ## **5. Deploy Workloads**

Apply a Kubernetes manifest:

```
kubectl apply -f deployment.yaml
```

---

## ## **6. Configure Load Balancing in EKS**

EKS supports:

* **Classic ELB**
* **Network Load Balancer (NLB)**
* **Application Load Balancer (ALB)** via AWS Load Balancer Controller

Install controller:

```
kubectl apply -f https://github.com/aws/eks-charts/.../aws-load-balancer-controller.yaml
```

---

## ## **7. Scaling EKS Nodes**

```
eksctl scale nodegroup --cluster=my-cluster --name=ng1 --nodes=5
```

---

## ## **8. Delete EKS Cluster**

```
eksctl delete cluster --name my-cluster
```

---

# 🎯 Summary

This document contains:

* Complete kubectl command references
* Resource types, short names, and usage
* Full EKS overview & setup steps

This should serve as a consolidated Kubernetes + EKS cheat sheet in markdown format.
