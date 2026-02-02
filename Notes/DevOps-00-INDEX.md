# DevOps Engineer's Complete Reference Guide
# Master Index & Quick Reference

---

## Complete Guide Overview

This comprehensive reference guide is designed to provide everything needed to become a FAANG-level DevOps/SRE engineer. The guide covers **17 parts** with over **20,000 lines** of documentation.

---

## Part Index

### Foundation (Parts 1-3)

| Part | Document | Description | Key Topics |
|------|----------|-------------|------------|
| 1 | [Linux Fundamentals](./DevOps-Complete-Reference-Guide-Part1-Linux.md) | Linux basics and shell scripting | File system, commands, shell scripting, text processing |
| 2 | [Linux Advanced](./DevOps-Complete-Reference-Guide-Part2-Linux-Advanced.md) | System administration | Users, permissions, systemd, packages, storage, performance |
| 3 | [Networking](./DevOps-Complete-Reference-Guide-Part3-Networking.md) | Network fundamentals | OSI/TCP-IP, IP addressing, DNS, firewalls, troubleshooting |

### Kubernetes & Cloud (Parts 4-5)

| Part | Document | Description | Key Topics |
|------|----------|-------------|------------|
| 4 | [Kubernetes](./DevOps-Complete-Reference-Guide-Part4-Kubernetes.md) | K8s architecture and objects | Pods, Deployments, Services, Ingress, Storage, RBAC, kubectl |
| 5 | [AKS & Azure](./DevOps-Complete-Reference-Guide-Part5-AKS-Azure.md) | Azure Kubernetes Service | AKS, ACR, VNet, Key Vault, Azure CLI, Workload Identity |

### CI/CD & Observability (Parts 6-7)

| Part | Document | Description | Key Topics |
|------|----------|-------------|------------|
| 6 | [Jenkins](./DevOps-Complete-Reference-Guide-Part6-Jenkins.md) | Jenkins CI/CD | Pipeline syntax, Kubernetes integration, shared libraries |
| 7 | [Observability](./DevOps-Complete-Reference-Guide-Part7-Observability-Apps.md) | Monitoring and troubleshooting | Prometheus, Logging, Tracing, Java/Node.js, troubleshooting |

### FAANG-Level Skills (Parts 8-17)

| Part | Document | Description | Key Topics |
|------|----------|-------------|------------|
| 8 | [Terraform](./DevOps-Complete-Reference-Guide-Part8-Terraform.md) | Infrastructure as Code | HCL, state management, modules, Azure provider, CI/CD |
| 9 | [System Design](./DevOps-Complete-Reference-Guide-Part9-SystemDesign.md) | Infrastructure architecture | CAP theorem, scaling, HA, caching, databases, microservices |
| 10 | [SRE Practices](./DevOps-Complete-Reference-Guide-Part10-SRE.md) | Site Reliability Engineering | SLIs/SLOs, error budgets, incident management, toil reduction |
| 11 | [GitOps](./DevOps-Complete-Reference-Guide-Part11-GitOps.md) | GitOps with ArgoCD & Flux | ArgoCD, Flux, secret management, progressive delivery |
| 12 | [Service Mesh](./DevOps-Complete-Reference-Guide-Part12-ServiceMesh.md) | Istio service mesh | Traffic management, mTLS, observability, resilience |
| 13 | [Security](./DevOps-Complete-Reference-Guide-Part13-Security.md) | Advanced security | Zero Trust, container security, supply chain, K8s security |
| 14 | [Chaos Engineering](./DevOps-Complete-Reference-Guide-Part14-ChaosEngineering.md) | Resilience testing | Chaos Mesh, Litmus, experiment design, GameDays |
| 15 | [Programming](./DevOps-Complete-Reference-Guide-Part15-Programming.md) | Python & Go for DevOps | Automation scripts, CLI tools, K8s client libraries |
| 16 | [Multi-Cloud](./DevOps-Complete-Reference-Guide-Part16-MultiCloud.md) | AWS & GCP essentials | EKS, GKE, cloud service comparison, CLI tools |
| 17 | [FinOps](./DevOps-Complete-Reference-Guide-Part17-FinOps.md) | Cost optimization | Azure Cost Management, Kubernetes costs, right-sizing |

### Complete Coverage (Parts 18-21)

| Part | Document | Description | Key Topics |
|------|----------|-------------|------------|
| 18 | [DSA for Interviews](./DevOps-Complete-Reference-Guide-Part18-DSA.md) | Data Structures & Algorithms | Arrays, Trees, Graphs, DP, LeetCode guide, patterns |
| 19 | [Message Queues](./DevOps-Complete-Reference-Guide-Part19-Messaging.md) | Event Streaming | Kafka, RabbitMQ, Azure Service Bus, Event Hubs |
| 20 | [Helm Deep Dive](./DevOps-Complete-Reference-Guide-Part20-Helm.md) | Kubernetes packaging | Chart development, templating, best practices |
| 21 | [Database Operations](./DevOps-Complete-Reference-Guide-Part21-Databases.md) | Database administration | PostgreSQL, MySQL, Redis, backup, HA, performance |

---

## Learning Path

### Recommended Study Order

```
Week 1-2: Linux & Networking Foundation
├── Part 1: Linux Fundamentals
├── Part 2: Linux Advanced
└── Part 3: Networking

Week 3-4: Kubernetes Core
├── Part 4: Kubernetes
├── Part 5: AKS & Azure
└── Part 20: Helm Deep Dive

Week 5-6: CI/CD & Observability
├── Part 6: Jenkins
└── Part 7: Observability

Week 7-8: Infrastructure as Code
├── Part 8: Terraform
└── Part 21: Database Operations

Week 9-10: System Design & SRE
├── Part 9: System Design
├── Part 10: SRE Practices
└── Part 19: Message Queues

Week 11-12: Advanced Topics
├── Part 11: GitOps
├── Part 12: Service Mesh
└── Part 13: Security

Week 13-14: Resilience & Programming
├── Part 14: Chaos Engineering
└── Part 15: Programming

Week 15-16: Cloud & Cost
├── Part 16: Multi-Cloud
└── Part 17: FinOps

Week 17-20: Interview Preparation
└── Part 18: DSA for Interviews (50-100 LeetCode problems)
```

---

## Quick Reference Cheat Sheets

### Linux Essential Commands

```bash
# File operations
ls -la                    # List all files
find /path -name "*.log"  # Find files
grep -r "pattern" /path   # Search in files

# Process management
ps aux                    # List processes
top / htop               # Monitor processes
kill -9 PID              # Force kill process

# System
systemctl status service  # Service status
journalctl -u service -f  # Service logs
df -h / free -h          # Disk/Memory usage

# Network
ss -tulpn                # Open ports
curl -v URL              # HTTP request
dig / nslookup domain    # DNS lookup
```

### Kubernetes Essential Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Pods
kubectl get pods -A
kubectl describe pod POD
kubectl logs POD -f
kubectl exec -it POD -- /bin/sh

# Deployments
kubectl get deployments
kubectl rollout status deployment/NAME
kubectl rollout undo deployment/NAME
kubectl scale deployment/NAME --replicas=5

# Debugging
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl debug POD --image=busybox
```

### Azure CLI Essential Commands

```bash
# Login and subscription
az login
az account set --subscription NAME

# AKS
az aks list -o table
az aks get-credentials --resource-group RG --name CLUSTER
az aks nodepool list --resource-group RG --cluster-name CLUSTER

# Resources
az resource list --resource-group RG -o table
az group list -o table
```

### Terraform Essential Commands

```bash
terraform init          # Initialize
terraform validate      # Validate config
terraform plan          # Preview changes
terraform apply         # Apply changes
terraform destroy       # Destroy resources
terraform state list    # List state
terraform import        # Import resource
```

### Git Essential Commands

```bash
git status              # Check status
git add .               # Stage all
git commit -m "msg"     # Commit
git push origin branch  # Push
git pull                # Pull changes
git log --oneline -10   # Recent commits
git diff                # Show changes
git stash / git stash pop  # Stash changes
```

### Docker Essential Commands

```bash
docker build -t name:tag .     # Build image
docker run -d -p 8080:80 name  # Run container
docker ps -a                   # List containers
docker logs CONTAINER          # View logs
docker exec -it CONTAINER sh   # Shell into container
docker images                  # List images
docker system prune -a         # Clean up
```

### Prometheus Query Language (PromQL)

```promql
# Request rate
rate(http_requests_total[5m])

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) /
sum(rate(http_requests_total[5m]))

# Latency percentile
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# CPU usage
(1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100

# Memory usage
container_memory_working_set_bytes / container_spec_memory_limit_bytes
```

### Kusto Query Language (KQL)

```kusto
// Container logs
ContainerLogV2
| where ContainerName == "myapp"
| where LogMessage contains "error"
| project TimeGenerated, LogMessage

// Pod status
KubePodInventory
| where Namespace == "production"
| summarize count() by PodStatus

// Performance
Perf
| where ObjectName == "Processor"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
```

---

## Interview Preparation Topics

### Technical Interview Focus Areas

```
1. LINUX & TROUBLESHOOTING
   ├── File systems and permissions
   ├── Process and memory management
   ├── Network configuration
   ├── Log analysis
   └── Performance troubleshooting

2. KUBERNETES
   ├── Architecture (control plane, nodes)
   ├── Pod lifecycle and scheduling
   ├── Networking (Services, Ingress, CNI)
   ├── Storage (PV, PVC, StorageClass)
   └── RBAC and security

3. CI/CD
   ├── Pipeline design
   ├── Testing strategies
   ├── Deployment strategies (blue-green, canary)
   ├── GitOps principles
   └── Artifact management

4. INFRASTRUCTURE AS CODE
   ├── Terraform state and modules
   ├── Best practices
   ├── Testing IaC
   └── CI/CD for infrastructure

5. SYSTEM DESIGN
   ├── Scalability patterns
   ├── High availability
   ├── Database design
   ├── Caching strategies
   └── Microservices

6. SRE
   ├── SLIs, SLOs, SLAs
   ├── Error budgets
   ├── Incident management
   ├── On-call best practices
   └── Postmortems

7. OBSERVABILITY
   ├── Metrics (Prometheus)
   ├── Logging (ELK, Fluent Bit)
   ├── Tracing (OpenTelemetry)
   └── Alerting strategies

8. SECURITY
   ├── Container security
   ├── Kubernetes security
   ├── Secrets management
   ├── Network policies
   └── Supply chain security

9. CODING
   ├── Python/Go basics
   ├── Automation scripts
   ├── API interactions
   └── LeetCode (Easy/Medium)
```

### Behavioral Interview Topics

```
1. Tell me about a complex outage you resolved
2. How do you prioritize competing priorities?
3. Describe a time you improved a system's reliability
4. How do you handle disagreements with team members?
5. Tell me about a time you automated a manual process
6. How do you stay current with technology?
7. Describe your approach to on-call
8. Tell me about a failure and what you learned
```

---

## Essential Books & Resources

### Must-Read Books

1. **Site Reliability Engineering** - Google (Free online)
2. **The Site Reliability Workbook** - Google (Free online)
3. **Designing Data-Intensive Applications** - Martin Kleppmann
4. **Kubernetes in Action** - Marko Lukša
5. **The Phoenix Project** - Gene Kim
6. **System Design Interview** - Alex Xu
7. **Infrastructure as Code** - Kief Morris

### Online Resources

- **Kubernetes**: kubernetes.io/docs
- **Terraform**: developer.hashicorp.com/terraform
- **Prometheus**: prometheus.io/docs
- **ArgoCD**: argo-cd.readthedocs.io
- **Azure**: learn.microsoft.com

---

## Document Statistics

| Part | Title | Approximate Lines |
|------|-------|-------------------|
| 1 | Linux Fundamentals | ~1,800 |
| 2 | Linux Advanced | ~2,000 |
| 3 | Networking | ~1,800 |
| 4 | Kubernetes | ~1,500 |
| 5 | AKS & Azure | ~1,300 |
| 6 | Jenkins | ~1,200 |
| 7 | Observability & Apps | ~1,200 |
| 8 | Terraform | ~2,000 |
| 9 | System Design | ~2,000 |
| 10 | SRE Practices | ~1,500 |
| 11 | GitOps | ~1,300 |
| 12 | Service Mesh | ~900 |
| 13 | Security | ~1,000 |
| 14 | Chaos Engineering | ~800 |
| 15 | Programming | ~1,200 |
| 16 | Multi-Cloud | ~800 |
| 17 | FinOps | ~1,000 |
| 18 | DSA for Interviews | ~1,500 |
| 19 | Message Queues | ~600 |
| 20 | Helm Deep Dive | ~400 |
| 21 | Database Operations | ~500 |
| - | Index | ~500 |
| **Total** | | **~25,000+ lines** |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Initial | Parts 1-7, original roadmap |
| 2.0 | Extended | Added Parts 8-17 for FAANG-level coverage |
| 3.0 | Complete | Added Parts 18-21: DSA, Messaging, Helm, Databases |

---

**This guide is your comprehensive reference for becoming an expert DevOps/SRE engineer. Study consistently, practice hands-on, and you will master these skills.**
