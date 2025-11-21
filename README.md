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


---


# HELM COMMANDS
# Basic interpretations/context
Chart:
```bash
It is the name of your chart in case it has been pulled and untarred.
It is <repo_name>/<chart_name> in case the repository has been added but chart not pulled.
It is the URL/Absolute path to the chart.
```
Name:
```bash 
It is the name you want to give to your current helm chart installation.
```
Release:
```bash
Is the name you assigned to an installation instance.
```
Revision:
```bash
Is the value from the Helm history command
```
Repo-name:
```bash
The name of a repository.
```
DIR:
```bash
Directory name/path
```

# Chart Management

```bash
helm create <name>                      # Creates a chart directory along with the common files and directories used in a chart.
helm package <chart-path>               # Packages a chart into a versioned chart archive file.
helm lint <chart>                       # Run tests to examine a chart and identify possible issues:
helm show all <chart>                   # Inspect a chart and list its contents:
helm show values <chart>                # Displays the contents of the values.yaml file
helm pull <chart>                       # Download/pull chart
helm pull <chart> --untar=true          # If set to true, will untar the chart after downloading it
helm pull <chart> --verify              # Verify the package before using it
helm pull <chart> --version <number>    # Default-latest is used, specify a version constraint for the chart version to use
helm dependency list <chart>            # Display a list of a chart’s dependencies:
```
# Install and Uninstall Apps

```bash
helm install <name> <chart>                           # Install the chart with a name
helm install <name> <chart> --namespace <namespace>   # Install the chart in a specific namespace
helm install <name> <chart> --set key1=val1,key2=val2 # Set values on the command line (can specify multiple or separate values with commas)
helm install <name> <chart> --values <yaml-file/url>  # Install the chart with your specified values
helm install <name> <chart> --dry-run --debug         # Run a test installation to validate chart (p)
helm install <name> <chart> --verify                  # Verify the package before using it
helm install <name> <chart> --dependency-update       # update dependencies if they are missing before installing the chart
helm uninstall <name>                                 # Uninstalls a release from the current (default) namespace
helm uninstall <release-name> --namespace <namespace> # Uninstalls a release from the specified namespace
```
# Perform App Upgrade and Rollback

```bash
helm upgrade <release> <chart>                            # Upgrade a release
helm upgrade <release> <chart> --rollback-on-failure      # If set, upgrade process rolls back changes made in case of failed upgrade.
helm upgrade <release> <chart> --dependency-update        # update dependencies if they are missing before installing the chart
helm upgrade <release> <chart> --version <version_number> # specify a version constraint for the chart version to use
helm upgrade <release> <chart> --values                   # specify values in a YAML file or a URL (can specify multiple)
helm upgrade <release> <chart> --set key1=val1,key2=val2  # Set values on the command line (can specify multiple or separate valuese)
helm upgrade <release> <chart> --force                    # Force resource updates through a replacement strategy
helm rollback <release> <revision>                        # Roll back a release to a specific revision
helm rollback <release> <revision>  --cleanup-on-fail     # Allow deletion of new resources created in this rollback when rollback fails
```
# List, Add, Remove, and Update Repositories

```bash 
helm repo add <repo-name> <url>   # Add a repository from the internet:
helm repo list                    # List added chart repositories
helm repo update                  # Update information of available charts locally from chart repositories
helm repo remove <repo_name>      # Remove one or more chart repositories
helm repo index <DIR>             # Read the current directory and generate an index file based on the charts found.
helm repo index <DIR> --merge     # Merge the generated index with an existing index file
helm search repo <keyword>        # Search repositories for a keyword in charts
helm search hub <keyword>         # Search for charts in the Artifact Hub or your own hub instance
```
# Helm Release monitoring

```bash
helm list                       # Lists all of the releases for a specified namespace, uses current namespace context if namespace not specified
helm list --all                 # Show all releases without any filter applied, can use -a
helm list --all-namespaces      # List releases across all namespaces, we can use -A
helm list -l key1=value1,key2=value2 # Selector (label query) to filter on, supports '=', '==', and '!='
helm list --date                # Sort by release date
helm list --deployed            # Show deployed releases. If no other is specified, this will be automatically enabled
helm list --pending             # Show pending releases
helm list --failed              # Show failed releases
helm list --uninstalled         # Show uninstalled releases (if 'helm uninstall --keep-history' was used)
helm list --superseded          # Show superseded releases
helm list -o yaml               # Prints the output in the specified format. Allowed values: table, json, yaml (default table)
helm status <release>           # This command shows the status of a named release.
helm status <release> --revision <number>   # if set, display the status of the named release with revision
helm history <release>          # Historical revisions for a given release.
helm env                        # Env prints out all the environment information in use by Helm.
```
# Download Release Information

```bash
helm get all <release>      # A human readable collection of information about the notes, hooks, supplied values, and generated manifest file of the given release.
helm get hooks <release>    # This command downloads hooks for a given release. Hooks are formatted in YAML and separated by the YAML '---\n' separator.
helm get manifest <release> # A manifest is a YAML-encoded representation of the Kubernetes resources that were generated from this release's chart(s). If a chart is dependent on other charts, those resources will also be included in the manifest.
helm get notes <release>    # Shows notes provided by the chart of a named release.
helm get values <release>   # Downloads a values file for a given release. use -o to format output
```
# Plugin Management

```bash
helm plugin install <path/url>      # Install plugins
helm plugin list                    # View a list of all installed plugins
helm plugin update <plugin>         # Update plugins
helm plugin uninstall <plugin>      # Uninstall a plugin
```
