# Troubleshooting Guides - Master Index

> **last_reviewed:** 2026-06-30

## Complete Troubleshooting Reference for DevOps/SRE

Every guide links back to learning notes. **Incident?** Start here. **Want depth?** Follow the learning reference link at the top of each guide.

---

## Folder Structure

```
Troubleshooting/
├── Linux/
├── Kubernetes/
├── Docker/
├── Networking/
├── Java/
├── NodeJS/
├── Jenkins/
├── Databases/          → PostgreSQL, MySQL, Redis + SQL Server Azure
├── Kafka/
└── Azure/
```

---

## Quick Access

| Technology | Guide | Learning notes |
|------------|-------|----------------|
| Linux | [System Troubleshooting](./Linux/01-Linux-Troubleshooting.md) | [Linux Fundamentals](../Notes/Linux/DevOps-01-Linux-Fundamentals.md) |
| Kubernetes | [K8s Troubleshooting](./Kubernetes/02-Kubernetes-Troubleshooting.md) | [Kubernetes](../Notes/Kubernetes/DevOps-04-Kubernetes.md) |
| Docker | [Container Troubleshooting](./Docker/03-Docker-Containers-Troubleshooting.md) | [Docker](../Notes/Docker/Docker-Complete-Guide.md) |
| Networking | [Network Troubleshooting](./Networking/04-Networking-Troubleshooting.md) | [Networking](../Notes/Networking/DevOps-03-Networking.md) |
| Java | [Java App Troubleshooting](./Java/05-Java-Application-Troubleshooting.md) | [Observability](../Notes/Observability/DevOps-07-Observability-Apps.md) |
| Node.js | [Node.js Troubleshooting](./NodeJS/06-NodeJS-Application-Troubleshooting.md) | [Observability](../Notes/Observability/DevOps-07-Observability-Apps.md) |
| Jenkins | [CI/CD Troubleshooting](./Jenkins/07-Jenkins-Troubleshooting.md) | [Jenkins](../Notes/Jenkins/DevOps-06-Jenkins-CICD.md) |
| Databases | [DB Troubleshooting](./Databases/08-Database-Troubleshooting.md) | [Databases](../Notes/Databases/DevOps-21-Databases.md) |
| SQL Server | [SQL Server / Azure SQL](./Databases/SQL-Server-Azure-Troubleshooting.md) | [SQL Server Azure](../Notes/Databases/SQL-Server-Azure.md) |
| Kafka | [Kafka Troubleshooting](./Kafka/Kafka-Troubleshooting.md) | [Kafka](../Notes/Kafka/Kafka-Complete-Guide.md) |
| Azure | [Cloud Troubleshooting](./Azure/) | [Azure Index](../Notes/Azure/Azure-00-INDEX.md) |

---

## Troubleshooting Methodology

### Level Classification

```
LEVEL 1 - DIRECT:        Error message clearly states the problem
LEVEL 2 - INTERMEDIATE:  Symptoms visible but cause needs investigation
LEVEL 3 - COMPLEX:       Symptoms misleading or hidden
```

### Systematic Approach

```
1. OBSERVE → 2. COLLECT → 3. CORRELATE → 4. HYPOTHESIZE
5. TEST → 6. RESOLVE → 7. PREVENT
```

---

## Quick Error Reference

### Exit Codes
| Code | Meaning |
|------|---------|
| 137 | OOMKilled / SIGKILL |
| 143 | SIGTERM |

### Kubernetes Pod States
| State | Common Causes |
|-------|---------------|
| Pending | Resources, node selector, PVC |
| ImagePullBackOff | Image name, auth, network |
| CrashLoopBackOff | App crash, bad config |

---

## Related Resources

- [Learning Notes](../Notes/INDEX.md)
- [Cheatsheets](../Notes/Cheatsheets/)
- [PetClinic runbooks](../Projects/PetClinic.md)
