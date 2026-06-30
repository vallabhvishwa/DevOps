# PetClinic Microservices — DevOps Capstone Project

> **last_reviewed:** 2026-06-30  
> **Goal:** Containerize [Spring PetClinic Microservices](https://github.com/spring-petclinic/spring-petclinic-microservices), deploy to AKS with Terraform + Helm + Jenkins, add observability, and document runbooks.

**Handbook links:** [Helm Enterprise](../Notes/Helm/Helm-Enterprise-Patterns.md) | [Terraform](../Notes/Terraform/DevOps-08-Terraform.md) | [Jenkins Groovy](../Notes/Jenkins/Jenkins-Groovy-Shared-Libraries.md) | [OpenTelemetry](../Notes/Observability/OpenTelemetry-Guide.md)

---

## What you are building

```
Developer push
      │
      ▼
┌─────────────┐    ┌──────┐    ┌─────┐    ┌──────────┐
│   Jenkins   │───▶│ ACR  │───▶│ AKS │───▶│ Grafana  │
│  CI/CD      │    │images│    │Helm │    │ alerts   │
└─────────────┘    └──────┘    └──────┘    └──────────┘
                      ▲
               ┌──────┴──────┐
               │  Terraform  │
               │  AKS+ACR+KV │
               └─────────────┘
```

**Services (upstream — you do not write Java):**
- API Gateway, Config Server, Discovery Server (Eureka)
- Customers, Vets, Visits services (+ optional GenAI)
- Each service has its own database

**Recommended forks:**
- Local learning: [spring-petclinic-microservices](https://github.com/spring-petclinic/spring-petclinic-microservices)
- K8s-native: [spring-petclinic-cloud](https://github.com/spring-petclinic/spring-petclinic-cloud)

---

## Repo layout (separate from this handbook)

```
petclinic-platform/
├── README.md
├── docs/
├── terraform/modules/
├── helm/petclinic/
├── pipelines/Jenkinsfile
└── scripts/
```

---

## Phase 0 — Understand the app (Week 1)

**Study:** [Docker](../Notes/Docker/Docker-Complete-Guide.md)

| Step | Action |
|------|--------|
| 1 | Clone upstream repo |
| 2 | `docker compose up` |
| 3 | Draw architecture diagram |
| 4 | Break one service — observe failure |

**Deliverable:** Architecture diagram  
**Verify:** UI via API Gateway; actuator health on each service

---

## Phase 1 — Containerize (Weeks 2–3)

**Study:** [Docker](../Notes/Docker/Docker-Complete-Guide.md) | [Azure Compute](../Notes/Azure/Azure-01-Compute-Services.md)

Build images, push to ACR, tag with git sha.

**Verify:** `az acr repository list --name <acr>`

---

## Phase 2 — Local Kubernetes (Weeks 4–5)

**Study:** [Kubernetes](../Notes/Kubernetes/DevOps-04-Kubernetes.md) | [Helm](../Notes/Helm/DevOps-20-Helm.md)

minikube/kind → manual YAML → Helm full stack → Ingress.

**Verify:** All pods Running; ingress responds

---

## Phase 3 — Terraform + AKS (Weeks 6–8)

**Study:** [Terraform](../Notes/Terraform/DevOps-08-Terraform.md) | [Private Link](../Notes/Azure/Azure-Private-Link-KeyVault-CICD.md)

AKS, ACR, Key Vault, managed identity.

**Verify:** `terraform apply` + `helm install` succeed

---

## Phase 4 — CI/CD (Weeks 9–10)

**Study:** [Jenkins](../Notes/Jenkins/DevOps-06-Jenkins-CICD.md) | [Groovy](../Notes/Jenkins/Jenkins-Groovy-Shared-Libraries.md)

Checkout → build → docker → push ACR → helm upgrade → smoke test.

---

## Phase 5 — Observability (Weeks 11–12)

**Study:** [Observability](../Notes/Observability/DevOps-07-Observability-Apps.md) | [OpenTelemetry](../Notes/Observability/OpenTelemetry-Guide.md)

Prometheus, Grafana, alerts on pod crash and 5xx.

---

## Phase 6 — Runbooks (Weeks 13–14)

**Study:** [SRE](../Notes/SRE/DevOps-10-SRE-Practices.md) | [Troubleshooting](../Troubleshooting/INDEX.md)

Document 5 failure scenarios with fix steps.

---

## 90-day checklist

- [ ] docker-compose runs locally
- [ ] Images in ACR
- [ ] Terraform + AKS
- [ ] Helm full stack
- [ ] CI/CD pipeline
- [ ] Grafana + alerts
- [ ] Runbook (5 scenarios)
- [ ] Public portfolio repo
