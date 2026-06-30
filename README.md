# DevOps & SRE Handbook

Personal knowledge base for upskilling from mid-level operations to senior DevOps / SRE — Linux through Azure AKS, CI/CD, IaC, observability, and production troubleshooting.

**Browse online:** [GitHub Pages](https://vallabhvishwa.github.io/DevOps/)  
**Repository:** https://github.com/vallabhvishwa/DevOps

---

## How to use this handbook

1. **New to DevOps** — Follow the [12-month roadmap](#12-month-roadmap); do labs, don't just read.
2. **Incident at work** — Go to [Troubleshooting](Troubleshooting/INDEX.md) first; link back to theory in Notes.
3. **Interview prep** — Foundations + [DSA subset](Notes/DSA/DSA-00-INDEX.md) + [PetClinic project](Projects/PetClinic.md).
4. **Daily reference** — Use [cheat sheets](Notes/Cheatsheets/).

### Local preview (Docsify)

```bash
git clone https://github.com/vallabhvishwa/DevOps.git
cd DevOps
npx docsify serve . -p 3000
# Open http://localhost:3000
```

---

## Structure

| Folder | Purpose |
|--------|---------|
| [Notes/](Notes/INDEX.md) | Learning guides by topic |
| [Troubleshooting/](Troubleshooting/INDEX.md) | Production runbooks and debug flows |
| [Projects/](Projects/PetClinic.md) | End-to-end capstone (PetClinic on AKS) |
| [Notes/Cheatsheets/](Notes/Cheatsheets/) | One-page quick references |

---

## 12-month roadmap

| Month | Study (Notes) | Build (PetClinic) | Cert (optional) |
|-------|---------------|-------------------|-----------------|
| 1 | Linux, Networking, [Scripting](Notes/Scripting/Scripting-Automation-Guide.md) | Phase 0: run app locally | — |
| 2 | [Docker](Notes/Docker/Docker-Complete-Guide.md), [Git](Notes/Git/Git-Advanced-Guide.md) | Phase 1: images to ACR | — |
| 3–4 | [Kubernetes](Notes/Kubernetes/DevOps-04-Kubernetes.md), [Helm](Notes/Helm/DevOps-20-Helm.md) | Phase 2: local K8s deploy | CKA prep |
| 5–6 | [Azure](Notes/Azure/Azure-00-INDEX.md), [Terraform](Notes/Terraform/DevOps-08-Terraform.md) | Phase 3: AKS + IaC | AZ-104 prep |
| 7 | [Jenkins](Notes/Jenkins/DevOps-06-Jenkins-CICD.md), [Groovy shared libs](Notes/Jenkins/Jenkins-Groovy-Shared-Libraries.md) | Phase 4: CI/CD pipeline | — |
| 8 | [Observability](Notes/Observability/DevOps-07-Observability-Apps.md), [OpenTelemetry](Notes/Observability/OpenTelemetry-Guide.md) | Phase 5: metrics and alerts | — |
| 9 | [SRE](Notes/SRE/DevOps-10-SRE-Practices.md), [Incident Management](Notes/Incident-Management/Incident-Management-Guide.md) | Phase 6: runbooks | — |
| 10 | [Security](Notes/Security/DevOps-13-Security.md), [Private Link + Key Vault](Notes/Azure/Azure-Private-Link-KeyVault-CICD.md) | Harden PetClinic | — |
| 11 | [GitOps](Notes/GitOps/DevOps-11-GitOps.md), [Platform Engineering](Notes/Platform-Engineering/Platform-Engineering-Guide.md) | Optional: Argo CD | AZ-400 prep |
| 12 | [System Design](Notes/System-Design/DevOps-09-System-Design.md), portfolio polish | README + demo | CKA / AZ-104 exam |

**Rule:** Every month, add a "Lessons learned" section to the relevant note from what broke in PetClinic or at work.

---

## Certification mapping

See [CERTIFICATIONS.md](CERTIFICATIONS.md) for CKA, CKAD, AZ-104, AZ-400, and Terraform Associate mapping to docs in this repo.

---

## Contributing (to yourself)

- Add `last_reviewed: YYYY-MM-DD` at the top of each doc when you update it.
- Log structural changes in [CHANGELOG.md](CHANGELOG.md).
- Notes = *why*; Troubleshooting = *what to run when it breaks*.

---

## License

Personal reference — use and adapt freely.
