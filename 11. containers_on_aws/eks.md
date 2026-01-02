<!-- Here is an in-depth yet focused explanation of AWS EKS (Elastic Kubernetes Service) tailored for the AWS Solutions Architect Associate (SAA-C03) exam — covering architecture, how it works, networking, security, integrations, pricing, and real-world use cases in a clear, structured, and note-friendly format.


---

🧠 What is AWS EKS?

Amazon Elastic Kubernetes Service (EKS) is a fully managed Kubernetes control plane by AWS.
It lets you run Kubernetes clusters on AWS without managing the control plane or etcd yourself.

✅ Kubernetes = container orchestration system
✅ EKS = AWS-managed Kubernetes platform


---

🚀 Why EKS?

Challenge	Solution

- Managing Kubernetes yourself is complex	EKS manages control plane, upgrades, high availability
- Scaling workloads securely across clusters	EKS integrates with IAM, VPC, ALB, etc.
- Hybrid/Multi-cloud consistency	Runs Kubernetes workloads in AWS or on-prem



---

⚙ Architecture Overview

```
EKS Architecture
 ┌────────────────────────┐
 │  EKS Control Plane     │  (Managed by AWS)
 │  • API Server          │
 │  • Scheduler           │
 │  • etcd (Cluster DB)   │
 └─────────▲──────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│  Node Group (EC2 or Fargate)            │
│  • kubelet runs here                    │    
│  • Actual containers/pods live here     │    
│  • IAM Roles for Service Accounts (IRSA)│    
└─────────────────────────────────────────┘

```

Control Plane is fully managed and deployed across 3 AZs for HA.

Worker Nodes (Data Plane) run containers in EC2 or Fargate.

kubectl CLI is used to manage EKS clusters.



---

🔄 How It Works (Step-by-Step Flow)

1. You create an EKS cluster (control plane is created by AWS).


2. Launch worker nodes (Node Groups):

Either EC2 instances (self-managed or managed node groups)

Or AWS Fargate (serverless compute for pods)



3. Install kubectl and configure aws-iam-authenticator.


4. Deploy workloads (pods, deployments, services, etc.) using Kubernetes YAML manifests.


5. EKS integrates with:

IAM for access control

VPC for networking

CloudWatch for logging

ALB/ELB/NLB for load balancing





---

🧱 Networking in EKS

Component	Details

VPC	Each cluster is launched inside a VPC. Subnets (private/public) are used.
ENI	Each pod gets an Elastic Network Interface (ENI) via VPC CNI plugin.
Security Groups	Applied to worker nodes or load balancers
Load Balancer Controller	Automatically provisions ALB/NLB for Kubernetes Ingress or Services


Pod networking models:

Amazon VPC CNI (default) – Each pod gets a native IP address from VPC subnet.

Alternative CNI like Calico can also be used.



---

🔐 Security in EKS

Feature	Description

IAM for Authentication	IAM users/roles can access EKS using aws-auth ConfigMap
RBAC for Authorization	Role-based access control within Kubernetes
IAM Roles for Service Accounts (IRSA)	Attach IAM permissions to Kubernetes service accounts
Pod Security Policies (deprecated)	Replaced by OPA/Gatekeeper/Kyverno
Secrets Management	Use Kubernetes Secrets or integrate with AWS Secrets Manager / KMS



---

💡 EKS Deployment Models

Model	Description

Managed Node Groups (EC2)	AWS provisions EC2 worker nodes for you
Self-Managed EC2	You manually provision and manage worker nodes
Fargate Profile	Serverless — pods run directly on Fargate, no EC2



---

🧪 Real-World Use Cases

Use Case	Description

Microservices	Deploy scalable, containerized microservices across nodes
Hybrid workloads	Use EKS Anywhere for on-prem clusters
Machine learning	Run ML workloads (e.g., TensorFlow, PyTorch) in containers
CI/CD Pipelines	Integrate with CodePipeline, ArgoCD, GitOps
Gaming	Scalable multiplayer game servers using Kubernetes pods



---

📊 Pricing Overview

You pay for:

1. Control Plane:

$0.10/hour per cluster (regardless of number of nodes)



2. Worker Nodes (EC2 or Fargate):

You pay for EC2/Fargate usage as normal

Additional costs for EBS, Load Balancers, NAT Gateway, etc.




> ❗ Unlike ECS (no control plane fee), EKS charges a fixed hourly fee for cluster operation.




---

🔗 Key Integrations

Service	Role

IAM	Access control via aws-auth
VPC	Networking, IP assignment
CloudWatch	Logs, metrics
ALB/NLB	Load balancing for services
ECR	Private container image registry
AWS WAF + Shield	Protection for Ingress endpoints
KMS	Encrypt secrets and etcd storage
EBS/EFS	Persistent volumes for stateful apps



---

🧠 EKS vs ECS (Exam Tip)

Feature	EKS (Kubernetes)	ECS

Orchestration	Kubernetes	AWS-native
Control Plane	You pay ($0.10/hr)	Free
Flexibility	Very high	AWS opinionated
Portability	Runs anywhere	AWS only (unless using ECS Anywhere)
Learning Curve	Higher	Lower



---

📌 Exam Tips (SAA-C03)

EKS control plane is multi-AZ and highly available by default

Use IAM Roles for Service Accounts (IRSA) to give pod-level permissions

EKS integrates with CloudWatch, ALB, ECR, and IAM

EKS worker nodes can be EC2 or Fargate

You are responsible for data plane security and updates (worker nodes)

Use EKS Add-ons to install common tools (e.g., CoreDNS, kube-proxy)



---

Would you like a one-page summary or YAML deployment flow next? -->



# **AWS EKS - SAA Notes**

## **📦 What is EKS?**
- **Managed Kubernetes** service on AWS
- AWS manages **control plane** (API server, etcd)
- You manage **worker nodes** (EC2 or Fargate)
- **CNCF-certified** Kubernetes

## **🏗️ Architecture Components**

### **1. EKS Control Plane**
- **Managed by AWS** in your account
- **Multi-AZ** for high availability
- **Automatically patched**
- **Free** (only pay for worker nodes + resources)

### **2. Worker Nodes**
Two options:

#### **EC2 Worker Nodes**
- **Self-managed** node groups
- **Managed node groups** (recommended)
- You choose instance types, scaling

#### **Fargate**
- **Serverless** containers
- No node management
- Pay per vCPU/memory
- **Limited** (no DaemonSets, privileged pods)

### **3. Networking**
- **VPC-native** using CNI plugin
- Each pod gets **real VPC IP**
- **Security groups** at pod level
- **Kube-proxy** for service networking

## **⚡ Key Features**

### **Managed Node Groups**
```yaml
# AWS handles:
- Node provisioning/scaling
- Updates/patches
- Node health
# You choose:
- Instance type
- AMI (EKS-optimized)
- Scaling config
```

### **IAM Integration**
- **IAM roles for service accounts** (IRSA)
- Pods assume IAM roles via OIDC
- **No need** for node instance profiles
- **Fine-grained** permissions per pod

### **Load Balancing**
- **ALB Ingress Controller** (recommended)
- **NLB** for TCP/UDP
- **Classic ELB** (legacy)
- Automatically provisions AWS LBs

## **🔐 Security**

### **1. Cluster Security**
- **Private endpoint** (recommended)
- **Public endpoint** with CIDR restrictions
- **Encrypted secrets** with KMS
- **Audit logs** to CloudWatch

### **2. Network Security**
- **Security groups** for pods
- **Network policies** with Calico
- **VPC flow logs** monitoring
- **Private subnets** for nodes

## **🔄 Comparison: EKS vs ECS**

| Feature | **EKS** | **ECS** |
|---------|---------|---------|
| Orchestrator | Kubernetes | AWS proprietary |
| Portability | High (any K8s) | Low (AWS only) |
| Learning Curve | Steep | Easier |
| Community | Large (K8s) | Smaller |
| Cost | Control plane free | No control plane |

## **💡 When to Use EKS**

### **Choose EKS when:**
✅ Already using Kubernetes
✅ Need **multi-cloud** portability
✅ Large **K8s ecosystem** (Helm, operators)
✅ Team has **K8s expertise**

### **Choose ECS when:**
✅ AWS-only deployment
✅ Simpler container orchestration
✅ Less operational overhead
✅ Faster to get started

## **🔧 Setup Steps (Simplified)**
1. **Create EKS cluster** (control plane)
2. **Add node group** (EC2 or Fargate)
3. **Configure kubectl** (update kubeconfig)
4. **Deploy applications** (kubectl apply)
5. **Configure IAM** (IRSA for pods)

## **💰 Pricing**
- **Control plane**: Free
- **Worker nodes**: EC2/Fargate pricing
- **EBS volumes** for persistent storage
- **Load balancers** if used
- **Data transfer** costs

## **🎯 Exam Focus Points**
1. **Managed vs self-managed nodes**
2. **Fargate limitations** (no DaemonSets, privileged pods)
3. **IRSA** for pod permissions
4. **ALB Ingress Controller** for HTTP
5. **Private cluster endpoint** (security)
6. **EKS vs ECS** use cases

## **🚀 Quick Facts**
- **Region availability**: All major regions
- **Kubernetes versions**: Supports recent 3 versions
- **Maintenance**: AWS patches control plane
- **Compliance**: HIPAA, PCI, SOC compliant
- **Integration**: Works with AWS services (RDS, S3, etc.)

---

**TL;DR:** EKS = AWS-managed Kubernetes. Use if you need K8s features/portability. Otherwise, ECS is simpler.