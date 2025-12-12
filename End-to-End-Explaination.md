Great — here’s a focused end-to-end explanation of how the pieces fit together in this repo (EKS + Ingress + ALB) and what happens when you deploy the three-tier app.

High-level flow (end-to-end)
- You build Docker images for frontend, backend and push them to ECR.
- You create an EKS cluster (control plane managed by AWS) and worker nodes (nodegroup or Fargate).
- You use Helm charts in this repo to deploy:
  - MongoDB (Stateful PV, Service)
  - Backend (Deployment, Service)
  - Frontend (Deployment, Service)
- You enable an Ingress resource (IngressClass "alb") in the Helm values. The AWS Load Balancer Controller watches the Ingress resources and creates an AWS Application Load Balancer (ALB).
- The ALB receives internet traffic and forwards requests to target groups that contain your Pod IPs (target-type=ip) according to host/path rules (e.g., "/" -> frontend, "/api" -> backend).
- Clients use the ALB DNS name (or a CNAME to a custom domain) to access the app.

Step-by-step: what actually happens when you deploy (details)
1. Build & push images
   - Docker images are built locally and pushed to Amazon ECR. Helm values reference those image repositories and tags.

2. Helm deploys Kubernetes objects
   - Helm installs Deployments (pods), Services (typically ClusterIP), and an Ingress manifest (Ingress object) into namespace three-tier-app (per README).
   - Services expose your apps inside the cluster:
     - frontend Service -> pods serving UI (port 3000)
     - backend Service -> backend pods (port 3500)
     - mongodb Service -> DB pods (27017)

3. AWS Load Balancer Controller (ALB Ingress Controller)
   - The AWS Load Balancer Controller (installed into the cluster) watches Ingress objects that reference an IngressClass it supports (commonly "alb").
   - The controller needs AWS permissions — usually installed with IRSA (IAM Role for Service Account) so it can create ALBs, Target Groups, Security Groups, and manage listeners and rules.
   - When it sees an Ingress with className="alb" and relevant annotations, it:
     - Creates (or updates) an ALB in your VPC.
     - Creates Target Groups matching each backend defined in the Ingress.
     - Registers targets according to target-type:
       - target-type: ip — it registers Pod IPs (requires pod IPs to be routable in the VPC; AWS VPC CNI provides these).
       - target-type: instance — would route to node instance IPs (less common for pod-level routing).
     - Creates listeners and rules (HTTP/HTTPS) per the Ingress host/path rules and annotations.
     - Optionally configures TLS if you provide certificate ARN annotations.

4. ALB configuration and routing
   - Listener(s): ALB listens on ports configured in annotation (e.g., HTTP:80 or HTTPS:443).
   - Listener rules: host-based and path-based rules route traffic:
     - Example in this repo: host frontend.amanpathakdevops.study with paths "/" (frontend) and "/api" (backend).
     - The backend path can be explicitly linked to a k8s Service name and port in the Ingress (README shows setting backend service name/port for /api).
   - Target groups: health checks are configured (path/port) and the ALB marks targets healthy/unhealthy accordingly.
   - Target registration:
     - With target-type=ip, pods get registered directly (requires VPC CNI, pods must be in node subnets that ALB can reach).
   - Security groups: the controller will attach security groups to the ALB and configure rules so ALB can reach targets; node/pod networking also matters.

5. DNS
   - The ALB gets an AWS-generated DNS name. You either use this directly or create a CNAME from your domain (README suggests creating CNAME frontend.amanpathakdevops.study -> <ALB DNS>).

Key configuration pieces in this repo (what to look for)
- Ingress class and annotations (from README examples):
  - --set ingress.enabled=true
  - --set ingress.className="alb"
  - Annotations shown:
    - alb.ingress.kubernetes.io/scheme=internet-facing
    - alb.ingress.kubernetes.io/target-type=ip
    - alb.ingress.kubernetes.io/listen-ports='[{"HTTP":80}]'
  - Ingress hosts/paths:
    - host = frontend.amanpathakdevops.study
    - paths: "/" -> (frontend), "/api" -> (backend service name & port set explicitly)
  - For /api they set: --set ingress.hosts[0].paths[1].backend.service.name="backend-release-backend-chart" and port.number=3500

Important operational details & gotchas
- IRSA / IAM permissions: the aws-load-balancer-controller requires an IAM role that allows creation of ALBs, target groups, security groups, etc. Use IAM OIDC + ServiceAccount (IRSA) to scope permissions.
- Subnets & tagging: the controller will place ALB in subnets it detects (expects subnets tagged for the cluster or matching conditions). If subnets are not reachable or not properly tagged, ALB may fail to provision.
- target-type=ip requires pod IPs to be VPC-routable (usually true with AWS VPC CNI). If you use other CNI or hostNetwork settings, routes change.
- Health checks & readiness:
  - ALB health checks must match the pod HTTP path and port. If your backend readiness probe path is "/ok" (the README mentions an update), align ALB health check path or ensure backend responds on the service port for health checks.
  - If readiness probes fail, pods will be NotReady and ALB won’t route traffic to them.
- Security Groups / Network ACLs: ensure SG rules allow ALB -> node/pod traffic and allow inbound from ALB on your port(s). Node security group and ALB security group interactions are critical.
- TLS: To enable HTTPS, annotate the Ingress with certificate ARN (alb.ingress.kubernetes.io/certificate-arn) and add appropriate listener annotation for 443.
- Path precedence: prefix rules and exact matching — be mindful if you have both "/" and "/api" paths; Ingress controller/ALB handles priority but path specificity matters.

Verification and debugging checklist
- After deploying ingress:
  - kubectl get ingress -n three-tier-app  (shows ALB hostname once created)
  - kubectl describe ingress <name> -n three-tier-app (verify rules & backend refs)
  - kubectl get svc -n three-tier-app (check services & ports)
  - kubectl get pods -n three-tier-app (check pod readiness)
  - kubectl logs -n kube-system deployment/aws-load-balancer-controller (controller logs)
- In AWS:
  - AWS Console → EC2 → Load Balancers: find ALB, check listeners and rules.
  - AWS Console → Target Groups: check targets and Target Health.
  - aws elbv2 describe-target-health --target-group-arn <arn>
- Common failures:
  - ALB stuck in provisioning: check controller logs for error (often IAM or subnet/tagging issue).
  - Targets show unhealthy: check pod readiness/liveness and ALB health-check path/port.
  - 403/404 from ALB: check listener rules and path-to-service mapping; ensure Ingress configured correct backend.service.name and port.

Why the ALB approach is used here
- ALB provides application-layer (HTTP/HTTPS) routing with host/path rules, TLS termination, and advanced features (redirects, WAF integration).
- AWS Load Balancer Controller translates Kubernetes Ingress into ALB configuration so you can manage routing with k8s manifests and Helm values.

Quick mapping: objects to AWS resources
- Kubernetes Ingress (class alb, annotations) -> AWS ALB + listeners + rules
- Kubernetes Service (ClusterIP) targeted by Ingress -> ALB Target Group that routes to Pod IPs
- Deployment -> Pods -> registered targets in Target Group
- aws-load-balancer-controller (k8s deployment) -> AWS API actions to create/manage ALB resources

If you want, next I can:
- Walk through the exact Ingress resource generated by the Helm chart from this repo (show YAML).
- List the typical IAM policy for the aws-load-balancer-controller and the IRSA ServiceAccount example.
- Show a short troubleshooting flow for a specific error you’re seeing (e.g., ALB not created, targets unhealthy, 404s).

Which of those would help you most right now?
