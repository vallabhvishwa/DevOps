# Troubleshooting Guides - Master Index
## Complete Troubleshooting Reference for DevOps/SRE

---

## Folder Structure

```
Troubleshooting/
├── Linux/        → System, processes, permissions, disk, network
├── Kubernetes/   → Pods, services, storage, workloads
├── Docker/       → Containers, images, builds, networking
├── Networking/   → Connectivity, DNS, firewall, SSL/TLS
├── Java/         → JVM, memory, GC, startup, containers
├── NodeJS/       → V8, event loop, memory, dependencies
├── Jenkins/      → Builds, pipelines, K8s agents
├── Databases/    → PostgreSQL, MySQL, Redis
└── Azure/        → VMs, AKS, Storage, SQL, Identity
```

---

## Quick Access

| Technology | Guide | Key Scenarios |
|------------|-------|---------------|
| [Linux](./Linux/) | System Troubleshooting | CPU, Memory, Disk, Network, Permissions |
| [Kubernetes](./Kubernetes/) | K8s Troubleshooting | Pod States, Scheduling, Networking, Storage |
| [Docker](./Docker/) | Container Troubleshooting | Startup, Images, Builds, Resources |
| [Networking](./Networking/) | Network Troubleshooting | Connectivity, DNS, Firewall, SSL |
| [Java](./Java/) | Java App Troubleshooting | JVM, Memory, GC, Performance |
| [NodeJS](./NodeJS/) | Node.js Troubleshooting | V8, Event Loop, Memory Leaks |
| [Jenkins](./Jenkins/) | CI/CD Troubleshooting | Builds, Pipelines, K8s Agents |
| [Databases](./Databases/) | Database Troubleshooting | PostgreSQL, MySQL, Redis |
| [Azure](./Azure/) | Cloud Troubleshooting | VMs, AKS, Storage, Networking |

---

## Troubleshooting Methodology

### Level Classification

```
LEVEL 1 - DIRECT:        Error message clearly states the problem
                         Solution is straightforward

LEVEL 2 - INTERMEDIATE:  Symptoms visible but cause needs investigation
                         Requires connecting multiple data points

LEVEL 3 - COMPLEX:       Symptoms misleading or hidden
                         Requires deep analysis and theory knowledge
```

### Systematic Approach

```
1. OBSERVE        What exactly is happening?
2. COLLECT        Gather logs, metrics, events
3. CORRELATE      Connect symptoms to possible causes
4. HYPOTHESIZE    Form theories based on evidence
5. TEST           Validate theories with targeted checks
6. RESOLVE        Fix root cause, not just symptoms
7. PREVENT        Add monitoring/alerting for future
```

---

## Quick Error Reference

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 126 | Permission denied (not executable) |
| 127 | Command not found |
| 137 | Killed (OOMKilled, SIGKILL) |
| 139 | Segmentation fault |
| 143 | Terminated (SIGTERM) |

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

### Kubernetes Pod States
| State | Common Causes |
|-------|---------------|
| Pending | No resources, node selector, taints |
| ImagePullBackOff | Wrong image, auth failed, network |
| CrashLoopBackOff | App crash, bad config, missing deps |
| Running but not Ready | Probe failing, app not ready |
| Terminating | Finalizers, stuck processes |

---

## Related Resources

- [Learning Notes](../Notes/INDEX.md)
