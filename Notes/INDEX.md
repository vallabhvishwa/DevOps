# DevOps Learning Notes - Master Index

> **last_reviewed:** 2026-06-30

## Complete Knowledge Base for Senior DevOps/SRE Engineers

---

## Start here

| Resource | Purpose |
|----------|---------|
| [Handbook README](../README.md) | 12-month roadmap |
| [PetClinic Project](../Projects/PetClinic.md) | Hands-on capstone |
| [Certifications](../CERTIFICATIONS.md) | Exam ↔ doc mapping |
| [Cheatsheets](./Cheatsheets/) | One-page references |
| [Troubleshooting](../Troubleshooting/INDEX.md) | Production runbooks |

---

## Folder Structure

```
Notes/
├── Cheatsheets/            → Quick references (Linux, kubectl, Terraform, Helm, Azure)
├── Linux/                  → Linux Fundamentals & Advanced
├── Networking/             → TCP/IP, DNS, Firewalls, VPN
├── Docker/                 → Images, Containers, Compose
├── Kubernetes/             → K8s Core + AKS Specifics
├── Azure/                  → All Azure Services + Private Link / Key Vault
├── Jenkins/                → CI/CD + Groovy Shared Libraries
├── Terraform/              → Infrastructure as Code
├── Helm/                   → Charts + Enterprise Patterns
├── Git/                    → Advanced Git & Workflows
├── Observability/          → Monitoring, Logging, OpenTelemetry
├── Platform-Engineering/   → IDP, golden paths
├── SRE/                    → SRE Practices & Reliability
├── GitOps/                 → ArgoCD, Flux, Git Workflows
├── Service-Mesh/           → Istio, Traffic Management
├── Security/               → DevSecOps, Container Security
├── SSL-TLS/                → Certificates & Encryption
├── Chaos-Engineering/      → Resilience Testing
├── Programming/            → Python, Go for DevOps
├── Multi-Cloud/            → AWS, GCP Comparison
├── FinOps/                 → Cost Optimization
├── Messaging/              → Kafka, RabbitMQ, Queues
├── Databases/              → PostgreSQL, MySQL, Redis, SQL Server Azure
├── System-Design/          → Architecture Patterns
├── Performance-Testing/    → k6, JMeter, Load Testing
├── Incident-Management/    → On-Call, Response, Post-Mortems
├── Deployment-Strategies/  → Blue-Green, Canary, Feature Flags
├── Scripting/              → Bash, PowerShell Automation
└── DSA/                    → Interview subset (4 parts)
```

---

## Learning Path (Recommended Order)

```
WEEK 1-2:   Linux → Networking → Scripting
WEEK 3:     Docker
WEEK 4-5:   Kubernetes → Helm (+ Enterprise Patterns)
WEEK 6-7:   Azure (all guides)
WEEK 8:     Jenkins → Groovy Shared Libraries
WEEK 9:     Terraform
WEEK 10:    Observability → OpenTelemetry → SRE
WEEK 11:    Security → Private Link/Key Vault → Service-Mesh
WEEK 12:    Databases (incl. SQL Server) → Messaging
ONGOING:    PetClinic project → Platform Engineering → DSA subset
```

---

## Quick Access by Category

### Foundation
| Folder | Content |
|--------|---------|
| [Linux](./Linux/) | System administration, processes |
| [Networking](./Networking/) | TCP/IP, DNS, firewalls |
| [Cheatsheets](./Cheatsheets/) | Linux, kubectl, Terraform, Helm, Azure CLI |

### Container & Orchestration
| Folder | Content |
|--------|---------|
| [Kubernetes](./Kubernetes/) | Pods, Services, Deployments, AKS |
| [Helm](./Helm/) | Charts, enterprise multi-env patterns |
| [Service-Mesh](./Service-Mesh/) | Istio, mTLS |

### Cloud Platform
| Folder | Content |
|--------|---------|
| [Azure](./Azure/) | Compute through Load Balancers + Private Link |

### CI/CD & Infrastructure
| Folder | Content |
|--------|---------|
| [Jenkins](./Jenkins/) | Pipelines + Groovy shared libraries |
| [Terraform](./Terraform/) | HCL, state, modules |
| [GitOps](./GitOps/) | ArgoCD, Flux |

### Reliability & Operations
| Folder | Content |
|--------|---------|
| [Observability](./Observability/) | Prometheus, Grafana, OpenTelemetry |
| [SRE](./SRE/) | SLOs, error budgets |
| [Platform Engineering](./Platform-Engineering/) | IDP, golden paths |

### Data
| Folder | Content |
|--------|---------|
| [Databases](./Databases/) | PG/MySQL/Redis + [SQL Server Azure](./Databases/SQL-Server-Azure.md) |
| [Messaging](./Messaging/) | Kafka, queues |

---

## Related Resources

- [Troubleshooting Guides](../Troubleshooting/INDEX.md)
- [AKS Learning Roadmap](./DevOps-AKS-Learning-Roadmap.md)
- [Changelog](../CHANGELOG.md)
