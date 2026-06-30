# Certification Mapping

> **last_reviewed:** 2026-06-30

Maps certification exam domains to documents in this handbook. Use alongside [PetClinic project](Projects/PetClinic.md) for hands-on practice.

---

## CKA — Certified Kubernetes Administrator

| Exam domain | Handbook docs | Hands-on |
|-------------|-----------------|----------|
| Cluster architecture | [Kubernetes](Notes/Kubernetes/DevOps-04-Kubernetes.md), [AKS](Notes/Kubernetes/DevOps-05-AKS-Azure.md) | PetClinic Phase 2–3 |
| Workloads & scheduling | [Kubernetes](Notes/Kubernetes/DevOps-04-Kubernetes.md) | Deployments, probes, resources |
| Services & networking | [Networking](Notes/Networking/DevOps-03-Networking.md), [Service Mesh](Notes/Service-Mesh/DevOps-12-Service-Mesh.md) | Ingress, NetworkPolicy |
| Storage | [Kubernetes](Notes/Kubernetes/DevOps-04-Kubernetes.md) | PVC, StorageClass |
| Troubleshooting | [K8s Troubleshooting](Troubleshooting/Kubernetes/02-Kubernetes-Troubleshooting.md) | [kubectl Cheatsheet](Notes/Cheatsheets/kubectl-Cheatsheet.md) |

**Cheat sheet:** [kubectl-Cheatsheet.md](Notes/Cheatsheets/kubectl-Cheatsheet.md)

---

## CKAD — Certified Kubernetes Application Developer

| Exam domain | Handbook docs |
|-------------|----------------|
| Core concepts | [Kubernetes](Notes/Kubernetes/DevOps-04-Kubernetes.md) |
| Configuration | [Helm](Notes/Helm/DevOps-20-Helm.md), [Helm Enterprise](Notes/Helm/Helm-Enterprise-Patterns.md) |
| Multi-container pods | [Docker](Notes/Docker/Docker-Complete-Guide.md) |
| Observability | [Observability](Notes/Observability/DevOps-07-Observability-Apps.md) |
| Pod design | [Deployment Strategies](Notes/Deployment-Strategies/Deployment-Strategies-Guide.md) |

---

## AZ-104 — Azure Administrator

| Exam domain | Handbook docs |
|-------------|----------------|
| Identity & governance | [Azure Identity](Notes/Azure/Azure-05-Identity-Security.md) |
| Storage | [Azure Storage](Notes/Azure/Azure-03-Storage-Services.md) |
| Compute | [Azure Compute](Notes/Azure/Azure-01-Compute-Services.md), [AKS](Notes/Kubernetes/DevOps-05-AKS-Azure.md) |
| Networking | [Azure Networking](Notes/Azure/Azure-02-Networking-Services.md), [Load Balancers](Notes/Azure/Azure-10-Load-Balancers.md), [Private Link](Notes/Azure/Azure-Private-Link-KeyVault-CICD.md) |
| Monitoring | [Azure Monitoring](Notes/Azure/Azure-06-Monitoring-Management.md) |

**Cheat sheet:** [Azure-CLI-Cheatsheet.md](Notes/Cheatsheets/Azure-CLI-Cheatsheet.md)

---

## AZ-400 — Azure DevOps Engineer Expert

| Exam domain | Handbook docs | Project |
|-------------|----------------|---------|
| CI/CD | [Jenkins](Notes/Jenkins/DevOps-06-Jenkins-CICD.md), [Groovy](Notes/Jenkins/Jenkins-Groovy-Shared-Libraries.md), [Azure DevOps](Notes/Azure/Azure-07-DevOps-CICD.md) | PetClinic Phase 4 |
| IaC | [Terraform](Notes/Terraform/DevOps-08-Terraform.md) | PetClinic Phase 3 |
| Source control | [Git](Notes/Git/Git-Advanced-Guide.md) | — |
| Security | [Security](Notes/Security/DevOps-13-Security.md), [Key Vault CI/CD](Notes/Azure/Azure-Private-Link-KeyVault-CICD.md) | PetClinic Phase 7 |
| Monitoring | [Observability](Notes/Observability/DevOps-07-Observability-Apps.md), [SRE](Notes/SRE/DevOps-10-SRE-Practices.md) | PetClinic Phase 5 |

---

## HashiCorp Terraform Associate

| Exam domain | Handbook docs |
|-------------|----------------|
| IaC concepts | [Terraform](Notes/Terraform/DevOps-08-Terraform.md) |
| Terraform workflow | [Terraform](Notes/Terraform/DevOps-08-Terraform.md) — plan, apply, state |
| Modules | [Terraform](Notes/Terraform/DevOps-08-Terraform.md), PetClinic `terraform/modules/` |
| State & backends | [Terraform](Notes/Terraform/DevOps-08-Terraform.md) |

**Cheat sheet:** [Terraform-Cheatsheet.md](Notes/Cheatsheets/Terraform-Cheatsheet.md)

---

## Study order recommendation

```
Month 1–4:  Foundations + CKA prep + PetClinic Phases 0–2
Month 5–6:  AZ-104 prep + PetClinic Phase 3
Month 7–8:  AZ-400 / Terraform Associate + PetClinic Phases 4–5
Month 9–12: CKA exam + portfolio + AZ-400 exam
```

---

## Related

- [12-month roadmap](README.md#12-month-roadmap)
- [DSA interview subset](Notes/DSA/DSA-00-INDEX.md) — coding interviews only
