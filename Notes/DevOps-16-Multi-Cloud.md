# DevOps Engineer's Complete Reference Guide
# Part 16: Multi-Cloud (AWS & GCP Basics)

---

## Table of Contents

1. [Multi-Cloud Strategy](#1-multi-cloud-strategy)
2. [AWS Core Services](#2-aws-core-services)
3. [GCP Core Services](#3-gcp-core-services)
4. [Cloud Service Comparison](#4-cloud-service-comparison)
5. [Multi-Cloud Tools](#5-multi-cloud-tools)
6. [AWS CLI Essentials](#6-aws-cli-essentials)
7. [GCP CLI Essentials](#7-gcp-cli-essentials)

---

## 1. Multi-Cloud Strategy

```
Why Multi-Cloud?
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  BENEFITS:                                                      │
│  ├── Avoid vendor lock-in                                       │
│  ├── Best-of-breed services                                     │
│  ├── Geographic coverage                                        │
│  ├── Disaster recovery                                          │
│  ├── Cost optimization                                          │
│  └── Compliance requirements                                    │
│                                                                 │
│  CHALLENGES:                                                    │
│  ├── Increased complexity                                       │
│  ├── Skills gap (different APIs, tools)                         │
│  ├── Security across clouds                                     │
│  ├── Networking between clouds                                  │
│  ├── Cost management complexity                                 │
│  └── Operational overhead                                       │
│                                                                 │
│  STRATEGIES:                                                    │
│  ├── Kubernetes-first: Use K8s as abstraction layer             │
│  ├── Terraform: IaC across all clouds                           │
│  ├── Cloud-agnostic tools: GitLab, Jenkins, Datadog             │
│  └── Portable applications: Containers, microservices           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. AWS Core Services

### 2.1 AWS Service Overview

```
AWS Essential Services for DevOps:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  COMPUTE:                                                       │
│  ├── EC2          - Virtual machines                            │
│  ├── ECS          - Container orchestration                     │
│  ├── EKS          - Managed Kubernetes                          │
│  ├── Lambda       - Serverless functions                        │
│  └── Fargate      - Serverless containers                       │
│                                                                 │
│  STORAGE:                                                       │
│  ├── S3           - Object storage                              │
│  ├── EBS          - Block storage for EC2                       │
│  ├── EFS          - Managed NFS                                 │
│  └── Glacier      - Archive storage                             │
│                                                                 │
│  NETWORKING:                                                    │
│  ├── VPC          - Virtual network                             │
│  ├── ALB/NLB      - Load balancers                              │
│  ├── Route 53     - DNS                                         │
│  ├── CloudFront   - CDN                                         │
│  └── Direct Connect - Dedicated connection                      │
│                                                                 │
│  DATABASE:                                                      │
│  ├── RDS          - Managed SQL databases                       │
│  ├── DynamoDB     - NoSQL                                       │
│  ├── ElastiCache  - Redis/Memcached                             │
│  └── Aurora       - High-perf MySQL/PostgreSQL                  │
│                                                                 │
│  SECURITY:                                                      │
│  ├── IAM          - Identity & access                           │
│  ├── KMS          - Key management                              │
│  ├── Secrets Manager - Secret storage                           │
│  └── WAF          - Web application firewall                    │
│                                                                 │
│  DEVOPS:                                                        │
│  ├── CodePipeline - CI/CD                                       │
│  ├── CodeBuild    - Build service                               │
│  ├── ECR          - Container registry                          │
│  └── CloudFormation - IaC                                       │
│                                                                 │
│  MONITORING:                                                    │
│  ├── CloudWatch   - Metrics, logs, alarms                       │
│  ├── X-Ray        - Distributed tracing                         │
│  └── CloudTrail   - API audit logging                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 EKS Overview

```
AWS EKS Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                      AWS Account                                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                         VPC                             │    │
│  │  ┌─────────────────────────────────────────────────────┐│    │
│  │  │            EKS Control Plane (AWS Managed)          ││    │
│  │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐               ││    │
│  │  │  │API Srvr │ │  etcd   │ │ Sched   │               ││    │
│  │  │  └─────────┘ └─────────┘ └─────────┘               ││    │
│  │  └─────────────────────────────────────────────────────┘│    │
│  │                          │                              │    │
│  │  ┌─────────────────────────────────────────────────────┐│    │
│  │  │            Node Groups (EC2 or Fargate)             ││    │
│  │  │                                                      ││    │
│  │  │  AZ-a           AZ-b           AZ-c                  ││    │
│  │  │  ┌─────┐       ┌─────┐       ┌─────┐                ││    │
│  │  │  │Node │       │Node │       │Node │                ││    │
│  │  │  │Node │       │Node │       │Node │                ││    │
│  │  │  └─────┘       └─────┘       └─────┘                ││    │
│  │  └─────────────────────────────────────────────────────┘│    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Key Differences from AKS:                                      │
│  ├── Node groups (vs. node pools)                               │
│  ├── AWS Load Balancer Controller (vs. Azure LB)                │
│  ├── IAM Roles for Service Accounts (IRSA)                      │
│  ├── eksctl CLI for cluster management                          │
│  └── VPC CNI plugin for networking                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# EKS Cluster creation
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 10 \
  --managed

# Get kubeconfig
aws eks update-kubeconfig --name my-cluster --region us-west-2

# Scale node group
eksctl scale nodegroup --cluster=my-cluster --name=workers --nodes=5
```

---

## 3. GCP Core Services

### 3.1 GCP Service Overview

```
GCP Essential Services for DevOps:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  COMPUTE:                                                       │
│  ├── Compute Engine    - Virtual machines                       │
│  ├── GKE               - Managed Kubernetes                     │
│  ├── Cloud Run         - Serverless containers                  │
│  ├── Cloud Functions   - Serverless functions                   │
│  └── App Engine        - PaaS                                   │
│                                                                 │
│  STORAGE:                                                       │
│  ├── Cloud Storage     - Object storage                         │
│  ├── Persistent Disk   - Block storage                          │
│  ├── Filestore         - Managed NFS                            │
│  └── Archive Storage   - Cold storage                           │
│                                                                 │
│  NETWORKING:                                                    │
│  ├── VPC               - Virtual network                        │
│  ├── Cloud Load Balancing - L4/L7 LB                            │
│  ├── Cloud DNS         - DNS                                    │
│  ├── Cloud CDN         - CDN                                    │
│  └── Cloud Interconnect - Dedicated connection                  │
│                                                                 │
│  DATABASE:                                                      │
│  ├── Cloud SQL         - Managed MySQL/PostgreSQL               │
│  ├── Cloud Spanner     - Global SQL                             │
│  ├── Firestore         - NoSQL                                  │
│  ├── Bigtable          - Wide-column NoSQL                      │
│  └── Memorystore       - Redis                                  │
│                                                                 │
│  SECURITY:                                                      │
│  ├── IAM               - Identity & access                      │
│  ├── Cloud KMS         - Key management                         │
│  ├── Secret Manager    - Secret storage                         │
│  └── Cloud Armor       - WAF/DDoS protection                    │
│                                                                 │
│  DEVOPS:                                                        │
│  ├── Cloud Build       - CI/CD                                  │
│  ├── Artifact Registry - Container/package registry             │
│  ├── Cloud Deploy      - Continuous delivery                    │
│  └── Deployment Manager - IaC (or use Terraform)                │
│                                                                 │
│  MONITORING:                                                    │
│  ├── Cloud Monitoring  - Metrics, dashboards                    │
│  ├── Cloud Logging     - Log management                         │
│  ├── Cloud Trace       - Distributed tracing                    │
│  └── Cloud Audit Logs  - API audit logging                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 GKE Overview

```
GKE Features:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  GKE MODES:                                                     │
│  ├── Standard: You manage nodes                                 │
│  └── Autopilot: Google manages nodes (per-pod billing)          │
│                                                                 │
│  KEY FEATURES:                                                  │
│  ├── Release channels (Rapid, Regular, Stable)                  │
│  ├── GKE Sandbox (gVisor for container isolation)               │
│  ├── Workload Identity (IAM for pods)                           │
│  ├── Binary Authorization (image signing)                       │
│  ├── Config Connector (K8s CRDs for GCP resources)              │
│  └── Multi-cluster Ingress                                      │
│                                                                 │
│  Differences from AKS:                                          │
│  ├── Node auto-provisioning (vs. explicit node pools)           │
│  ├── GKE Ingress (vs. NGINX/AGIC)                               │
│  ├── Workload Identity (similar to Workload Identity Fed)       │
│  ├── Container-native load balancing                            │
│  └── Anthos for multi-cluster management                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# GKE Cluster creation
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type e2-standard-4 \
  --enable-autoscaling --min-nodes 1 --max-nodes 10 \
  --workload-pool=PROJECT_ID.svc.id.goog

# GKE Autopilot
gcloud container clusters create-auto autopilot-cluster \
  --region us-central1

# Get kubeconfig
gcloud container clusters get-credentials my-cluster --zone us-central1-a

# Scale node pool
gcloud container clusters resize my-cluster --node-pool default-pool --num-nodes 5
```

---

## 4. Cloud Service Comparison

```
Service Comparison: Azure vs AWS vs GCP
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Category        Azure           AWS             GCP            │
│  ─────────────────────────────────────────────────────────────  │
│  Kubernetes      AKS             EKS             GKE            │
│  VMs             Virtual Machines EC2            Compute Engine │
│  Serverless      Functions       Lambda          Cloud Functions│
│  Container Svcs  Container Apps  Fargate         Cloud Run      │
│  Object Storage  Blob Storage    S3              Cloud Storage  │
│  Block Storage   Managed Disks   EBS             Persistent Disk│
│  File Storage    Azure Files     EFS             Filestore      │
│  SQL Database    Azure SQL       RDS             Cloud SQL      │
│  NoSQL           Cosmos DB       DynamoDB        Firestore      │
│  Cache           Azure Cache     ElastiCache     Memorystore    │
│  Virtual Network VNet            VPC             VPC            │
│  Load Balancer   Azure LB        ALB/NLB         Cloud LB       │
│  CDN             Azure CDN       CloudFront      Cloud CDN      │
│  DNS             Azure DNS       Route 53        Cloud DNS      │
│  IAM             Azure AD + RBAC IAM             IAM            │
│  Secret Mgmt     Key Vault       Secrets Manager Secret Manager │
│  Container Reg   ACR             ECR             Artifact Reg   │
│  CI/CD           Azure DevOps    CodePipeline    Cloud Build    │
│  Monitoring      Azure Monitor   CloudWatch      Cloud Monitor  │
│  Logging         Log Analytics   CloudWatch Logs Cloud Logging  │
│  Tracing         App Insights    X-Ray           Cloud Trace    │
│  IaC             ARM/Bicep       CloudFormation  Deployment Mgr │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Multi-Cloud Tools

```
Cloud-Agnostic Tools:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  INFRASTRUCTURE:                                                │
│  ├── Terraform     - IaC for all clouds                         │
│  ├── Pulumi        - IaC with programming languages             │
│  ├── Crossplane    - Kubernetes-native infrastructure           │
│  └── Ansible       - Configuration management                   │
│                                                                 │
│  KUBERNETES:                                                    │
│  ├── kubectl       - Works with any K8s                         │
│  ├── Helm          - Package manager                            │
│  ├── ArgoCD        - GitOps for any cluster                     │
│  ├── Istio/Linkerd - Service mesh                               │
│  └── Velero        - Backup/restore                             │
│                                                                 │
│  CI/CD:                                                         │
│  ├── Jenkins       - Self-hosted CI/CD                          │
│  ├── GitLab CI     - Git-based CI/CD                            │
│  ├── GitHub Actions - GitHub CI/CD                              │
│  └── CircleCI      - Cloud CI/CD                                │
│                                                                 │
│  MONITORING:                                                    │
│  ├── Prometheus    - Metrics collection                         │
│  ├── Grafana       - Dashboards                                 │
│  ├── Datadog       - Full observability                         │
│  ├── New Relic     - APM                                        │
│  └── Elastic Stack - Logs, APM                                  │
│                                                                 │
│  SECURITY:                                                      │
│  ├── HashiCorp Vault - Secrets management                       │
│  ├── Falco         - Runtime security                           │
│  ├── Trivy         - Vulnerability scanning                     │
│  └── OPA/Gatekeeper - Policy enforcement                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. AWS CLI Essentials

```bash
# Configure AWS CLI
aws configure
# Enter: Access Key ID, Secret Access Key, Region, Output format

# Identity
aws sts get-caller-identity

# EC2
aws ec2 describe-instances
aws ec2 describe-instances --query 'Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType}'
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# S3
aws s3 ls
aws s3 ls s3://my-bucket/
aws s3 cp file.txt s3://my-bucket/
aws s3 sync ./local-dir s3://my-bucket/prefix/

# EKS
aws eks list-clusters
aws eks describe-cluster --name my-cluster
aws eks update-kubeconfig --name my-cluster --region us-west-2

# ECR
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com
aws ecr describe-repositories
aws ecr list-images --repository-name my-app

# IAM
aws iam list-users
aws iam list-roles
aws iam get-role --role-name my-role

# Secrets Manager
aws secretsmanager list-secrets
aws secretsmanager get-secret-value --secret-id my-secret

# CloudWatch
aws logs describe-log-groups
aws logs tail /aws/eks/my-cluster/cluster --follow
```

---

## 7. GCP CLI Essentials

```bash
# Configure gcloud
gcloud init
gcloud auth login
gcloud config set project my-project

# Identity
gcloud auth list
gcloud config list

# Compute Engine
gcloud compute instances list
gcloud compute instances describe my-vm --zone us-central1-a
gcloud compute instances start my-vm --zone us-central1-a
gcloud compute instances stop my-vm --zone us-central1-a

# Cloud Storage
gsutil ls
gsutil ls gs://my-bucket/
gsutil cp file.txt gs://my-bucket/
gsutil rsync -r ./local-dir gs://my-bucket/prefix/

# GKE
gcloud container clusters list
gcloud container clusters describe my-cluster --zone us-central1-a
gcloud container clusters get-credentials my-cluster --zone us-central1-a
gcloud container node-pools list --cluster my-cluster --zone us-central1-a

# Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev
gcloud artifacts repositories list
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo

# IAM
gcloud iam service-accounts list
gcloud iam roles list --project my-project
gcloud projects get-iam-policy my-project

# Secret Manager
gcloud secrets list
gcloud secrets versions access latest --secret my-secret

# Cloud Logging
gcloud logging read "resource.type=gke_cluster" --limit 10
gcloud logging tail "resource.type=gke_cluster"
```

---

## Summary

Multi-cloud skills expand your capabilities:

1. **AWS Core**: EC2, S3, EKS, RDS, IAM, CloudWatch
2. **GCP Core**: Compute Engine, Cloud Storage, GKE, Cloud SQL, IAM
3. **Common Patterns**: Kubernetes is the unifying layer across clouds
4. **Cloud-Agnostic Tools**: Terraform, Prometheus, ArgoCD work everywhere
5. **CLI Proficiency**: aws, gcloud, az commands are similar in structure

Focus on understanding concepts—they translate across clouds. Once you know one cloud well, learning others is much faster.

---

**Next Part**: [Part 17: FinOps & Cost Optimization](./DevOps-Complete-Reference-Guide-Part17-FinOps.md)
