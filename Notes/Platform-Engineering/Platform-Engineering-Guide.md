# Platform Engineering — DevOps Guide

> **last_reviewed:** 2026-06-30  
> **See also:** [GitOps](../GitOps/DevOps-11-GitOps.md) | [SRE](../SRE/DevOps-10-SRE-Practices.md) | [FinOps](../FinOps/DevOps-17-FinOps.md)

Platform Engineering builds **internal developer platforms (IDPs)** so product teams ship faster without each team reinventing Kubernetes, CI/CD, and cloud plumbing.

---

## 1. DevOps vs Platform Engineering

| DevOps (traditional) | Platform Engineering |
|----------------------|----------------------|
| Embed in each team | Central platform team serves many teams |
| "You build it, you run it" | Golden paths + self-service |
| Custom Jenkins per project | Standardized pipelines as a service |
| Tribal knowledge | Documentation + templates + portals |

**Goal:** Reduce cognitive load — developers focus on app code; platform handles infra patterns.

---

## 2. Internal Developer Platform components

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Portal (Backstage)              │
│  Service catalog │ Docs │ Templates │ CI/CD status           │
└────────────────────────────┬────────────────────────────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     ▼                       ▼                       ▼
┌─────────┐           ┌─────────────┐          ┌─────────────┐
│ Golden  │           │  GitOps     │          │ Observability│
│ path    │           │  (Argo CD)  │          │ stack        │
│ templates│          │             │          │ (as a service)│
└─────────┘           └─────────────┘          └─────────────┘
     │                       │                       │
     └───────────────────────┴───────────────────────┘
                             ▼
                    ┌─────────────────┐
                    │  AKS / Cloud    │
                    └─────────────────┘
```

---

## 3. Golden paths

A **golden path** is the blessed, supported way to deploy a service.

**Example: "Deploy Java microservice to AKS"**

1. Run Backstage template → creates repo with Helm chart + Jenkinsfile
2. Push code → CI builds image → pushes ACR
3. GitOps repo updated → Argo CD syncs to AKS
4. Dashboards and alerts pre-configured

**What to standardize:**
- Helm chart base (labels, probes, resources, security context)
- CI pipeline stages
- Secret management (Key Vault + External Secrets)
- Ingress + TLS pattern
- Logging/metrics labels

Your [Helm Enterprise Patterns](../Helm/Helm-Enterprise-Patterns.md) and [Jenkins Groovy](../Jenkins/Jenkins-Groovy-Shared-Libraries.md) docs are building blocks for golden paths.

---

## 4. Service catalog

Track every service: owner, repo, on-call, dependencies, SLOs.

**Backstage** (`backstage.io`) — open source portal:
- `catalog-info.yaml` per service
- Links to Grafana, logs, runbooks
- Software templates (scaffolding)

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: customers-service
  description: PetClinic customers API
  annotations:
    github.com/project-slug: org/customers-service
spec:
  type: service
  lifecycle: production
  owner: team-billing
  system: petclinic
```

---

## 5. Self-service infrastructure

| Capability | Tooling |
|------------|---------|
| Provision env | Terraform + Terragrunt or Crossplane |
| Deploy app | Argo CD ApplicationSet |
| Secrets | External Secrets + Key Vault |
| Preview envs | Per-PR namespace + Helm |
| Cost visibility | FinOps tags + dashboards |

**Anti-pattern:** Ticket queue for every deploy — platform should enable self-service within guardrails.

---

## 6. Platform team responsibilities

```
BUILD:
  ├── Maintain golden path templates
  ├── Operate shared clusters (AKS)
  ├── CI/CD shared libraries
  └── Observability baseline

MEASURE:
  ├── DORA metrics (deploy frequency, lead time, MTTR, change fail %)
  ├── Developer satisfaction surveys
  └── Platform adoption rate

ITERATE:
  ├── Reduce toil (automate what teams ask twice)
  └── Deprecate unsupported paths
```

---

## 7. DORA metrics (what leadership cares about)

| Metric | Platform lever |
|--------|----------------|
| Deployment frequency | CI/CD automation, GitOps |
| Lead time for changes | Golden paths, less custom infra |
| Mean time to recovery | Runbooks, observability, chaos testing |
| Change failure rate | Canary deploys, automated rollback |

---

## 8. Getting started (practical)

**Month 1–2:** Document current state — how many ways do teams deploy today?  
**Month 3–4:** One golden path (e.g. Java → ACR → AKS via Helm)  
**Month 5–6:** Extract shared Jenkins library + base Helm chart  
**Month 7+:** Backstage catalog for top 10 services  

**Capstone:** [PetClinic](../../Projects/PetClinic.md) *is* a golden path prototype — treat it as your internal platform reference implementation.

---

## Related

- [GitOps](../GitOps/DevOps-11-GitOps.md)
- [Deployment Strategies](../Deployment-Strategies/Deployment-Strategies-Guide.md)
- [System Design](../System-Design/DevOps-09-System-Design.md)
- [Incident Management](../Incident-Management/Incident-Management-Guide.md)
