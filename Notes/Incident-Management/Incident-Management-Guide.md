# Incident Management - Complete Guide
## On-Call, Response, Post-Mortems, Runbooks

---

# 1. INCIDENT FUNDAMENTALS

## What Is an Incident?

```
INCIDENT = Unplanned interruption or reduction in quality of service

INCIDENT vs OTHER EVENTS:
┌─────────────────────────────────────────────────────────────────┐
│ Alert:      Automated notification of potential issue          │
│ Incident:   Confirmed impact on users/business                 │
│ Problem:    Root cause of one or more incidents                │
│ Change:     Planned modification to system                     │
└─────────────────────────────────────────────────────────────────┘
```

## Severity Levels

```
SEVERITY CLASSIFICATION:
┌───────────┬─────────────────────────────────────────────────────┐
│ SEV 1     │ Complete outage, all users affected                │
│ (Critical)│ Revenue impact, data loss risk                     │
│           │ Response: Immediate, all hands                     │
│           │ Target: 15 min response, 1 hour resolution         │
├───────────┼─────────────────────────────────────────────────────┤
│ SEV 2     │ Major feature broken, many users affected          │
│ (High)    │ Significant business impact                        │
│           │ Response: Within 30 min                            │
│           │ Target: 4 hour resolution                          │
├───────────┼─────────────────────────────────────────────────────┤
│ SEV 3     │ Partial degradation, some users affected           │
│ (Medium)  │ Workaround available                               │
│           │ Response: Within 2 hours                           │
│           │ Target: 24 hour resolution                         │
├───────────┼─────────────────────────────────────────────────────┤
│ SEV 4     │ Minor issue, few users affected                    │
│ (Low)     │ No significant business impact                     │
│           │ Response: Next business day                        │
│           │ Target: Best effort                                │
└───────────┴─────────────────────────────────────────────────────┘
```

---

# 2. INCIDENT LIFECYCLE

```
INCIDENT LIFECYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Detection → Triage → Response → Resolution → Post-Mortem    │
│       │         │         │           │             │          │
│       ▼         ▼         ▼           ▼             ▼          │
│   Alerts    Classify   Diagnose   Fix issue    Document       │
│   Reports   Severity   Mitigate   Verify       Learn          │
│   Users     Assign     Escalate   Communicate  Improve        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 1: Detection

```
DETECTION SOURCES:
┌─────────────────────────────────────────────────────────────────┐
│ Monitoring Alerts:   Automated detection                       │
│ User Reports:        Support tickets, social media             │
│ Internal Reports:    Engineers notice issue                    │
│ External Reports:    Partners, third-parties                   │
└─────────────────────────────────────────────────────────────────┘

KEY METRICS:
- MTTD (Mean Time To Detect): Time from issue start to detection
- Goal: Detect before users notice
```

## Phase 2: Triage

```
TRIAGE CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ☐ What is the impact? (users affected, revenue impact)        │
│ ☐ Is it ongoing or resolved?                                  │
│ ☐ Which systems are affected?                                 │
│ ☐ What changed recently?                                      │
│ ☐ Assign severity level                                       │
│ ☐ Assign incident commander                                   │
│ ☐ Create incident channel (Slack, Teams)                      │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 3: Response

```
INCIDENT RESPONSE STRUCTURE:

Incident Commander (IC):
- Coordinates response
- Makes decisions
- Manages communication
- Escalates as needed

Technical Lead:
- Diagnoses issue
- Implements fixes
- Coordinates technical team

Communications Lead:
- Updates stakeholders
- Writes status updates
- Manages customer communication

Scribe:
- Documents timeline
- Records actions taken
- Notes decisions made
```

## Phase 4: Resolution

```
RESOLUTION STEPS:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Implement fix or workaround                                 │
│ 2. Verify fix is working                                       │
│ 3. Monitor for recurrence                                      │
│ 4. Communicate resolution                                      │
│ 5. Determine if incident is truly resolved                     │
│ 6. Officially close incident                                   │
└─────────────────────────────────────────────────────────────────┘

KEY METRICS:
- MTTR (Mean Time To Resolve): Detection to resolution
- MTTA (Mean Time To Acknowledge): Detection to first response
```

## Phase 5: Post-Mortem

```
POST-MORTEM TEMPLATE:
┌─────────────────────────────────────────────────────────────────┐
│ INCIDENT SUMMARY                                               │
│ - Date/Time: 2024-01-15 14:30 - 16:45 UTC                     │
│ - Duration: 2 hours 15 minutes                                 │
│ - Severity: SEV 2                                              │
│ - Impact: 40% of users unable to login                        │
│                                                                 │
│ TIMELINE                                                        │
│ 14:30 - Deployment of auth-service v2.5.0                     │
│ 14:35 - First alerts fire for high error rate                 │
│ 14:40 - Incident declared, IC assigned                        │
│ 14:50 - Root cause identified (DB connection pool)            │
│ 15:00 - Rollback initiated                                     │
│ 15:15 - Rollback complete, errors decreasing                  │
│ 16:00 - Error rate back to normal                             │
│ 16:45 - Incident closed                                        │
│                                                                 │
│ ROOT CAUSE                                                      │
│ New version had misconfigured connection pool (10 vs 100)     │
│                                                                 │
│ CONTRIBUTING FACTORS                                            │
│ - Change not tested with production load                       │
│ - Config not in version control                                │
│                                                                 │
│ WHAT WENT WELL                                                  │
│ - Fast detection (5 min)                                       │
│ - Clear rollback procedure                                     │
│                                                                 │
│ WHAT COULD BE IMPROVED                                          │
│ - Load testing before deployment                               │
│ - Config review in PR process                                  │
│                                                                 │
│ ACTION ITEMS                                                    │
│ 1. Add load testing to CI/CD - Owner: @john - Due: Jan 30     │
│ 2. Move config to version control - Owner: @jane - Due: Jan 22│
│ 3. Add connection pool metric to dashboard - Owner: @bob      │
└─────────────────────────────────────────────────────────────────┘
```

---

# 3. ON-CALL PRACTICES

## On-Call Structure

```
ON-CALL ROTATION:
┌─────────────────────────────────────────────────────────────────┐
│ Primary On-Call:    First responder                            │
│ Secondary On-Call:  Backup if primary unavailable              │
│ Manager On-Call:    Escalation for major incidents             │
│                                                                 │
│ Rotation Schedule:                                              │
│ - Weekly rotations common                                      │
│ - Handoff at consistent time (e.g., Monday 9am)                │
│ - Follow the sun for global teams                              │
└─────────────────────────────────────────────────────────────────┘
```

## On-Call Expectations

```
ON-CALL RESPONSIBILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Respond to alerts within SLA (usually 15 min)               │
│ ✓ Have laptop and reliable internet access                    │
│ ✓ Be within phone reception                                   │
│ ✓ Avoid activities that prevent response (flights, etc.)      │
│ ✓ Escalate when needed (don't be a hero)                      │
│ ✓ Document actions taken                                       │
│ ✓ Hand off clearly at rotation end                            │
└─────────────────────────────────────────────────────────────────┘

ON-CALL COMPENSATION:
- Additional pay for on-call shifts
- Time off after incidents
- Fair rotation distribution
```

## Escalation Paths

```
ESCALATION MATRIX:
┌─────────────────────────────────────────────────────────────────┐
│ Level 1: Primary on-call engineer                              │
│          ↓ (15 min no response or needs help)                  │
│ Level 2: Secondary on-call / Team lead                         │
│          ↓ (30 min or SEV 1)                                   │
│ Level 3: Engineering manager                                   │
│          ↓ (1 hour or business impact)                         │
│ Level 4: VP/Director + Exec communication                      │
└─────────────────────────────────────────────────────────────────┘
```

---

# 4. RUNBOOKS

## What Is a Runbook?

```
RUNBOOK = Step-by-step guide for handling specific scenarios

PURPOSE:
- Enable anyone on-call to handle known issues
- Reduce time to resolution
- Ensure consistent response
- Capture tribal knowledge
```

## Runbook Template

```markdown
# Runbook: High CPU on API Servers

## Summary
This runbook covers diagnosis and mitigation of high CPU alerts on API servers.

## Symptoms
- Alert: "API server CPU > 80% for 5 min"
- User impact: Slow API responses

## Prerequisites
- SSH access to API servers
- Access to monitoring dashboards
- Ability to restart services

## Diagnosis Steps

### 1. Check which process is consuming CPU
\```bash
ssh api-server-01
top -b -n 1 | head -20
\```

### 2. Check if it's garbage collection (Java)
\```bash
jstat -gc <pid> 1000 5
\```

### 3. Check for recent deployments
\```bash
kubectl rollout history deployment/api
\```

## Mitigation Steps

### Option A: Restart the service
\```bash
kubectl rollout restart deployment/api
\```
Wait 5 minutes and verify CPU returns to normal.

### Option B: Scale up temporarily
\```bash
kubectl scale deployment/api --replicas=5
\```

### Option C: Rollback recent deployment
\```bash
kubectl rollout undo deployment/api
\```

## Verification
- CPU alert clears
- API response times return to normal
- Check dashboard: [Link to dashboard]

## Escalation
If above steps don't resolve:
- Page secondary on-call
- Create incident ticket

## Related
- [GC Tuning Guide](#)
- [API Performance Dashboard](#)

## Changelog
- 2024-01-15: Added JVM GC check - @john
- 2023-11-20: Initial version - @jane
```

---

# 5. COMMUNICATION

## Status Page Updates

```
STATUS PAGE LEVELS:
┌─────────────────────────────────────────────────────────────────┐
│ Operational:         All systems normal                        │
│ Degraded:           Partial impact, service usable             │
│ Partial Outage:     Major feature unavailable                  │
│ Major Outage:       Service completely down                    │
└─────────────────────────────────────────────────────────────────┘

UPDATE TEMPLATE:
┌─────────────────────────────────────────────────────────────────┐
│ Investigating: We are investigating reports of [issue].       │
│                                                                 │
│ Identified: We have identified the cause of [issue].          │
│             [Brief description]. Working on fix.                │
│                                                                 │
│ Monitoring: A fix has been deployed. We are monitoring.       │
│                                                                 │
│ Resolved: The issue has been resolved. [Brief summary].       │
│           We apologize for any inconvenience.                   │
└─────────────────────────────────────────────────────────────────┘
```

## Internal Communication

```
INCIDENT CHANNEL FORMAT:
#incident-2024-01-15-api-outage

OPENING MESSAGE:
┌─────────────────────────────────────────────────────────────────┐
│ 🚨 INCIDENT DECLARED                                           │
│                                                                 │
│ Summary: API returning 500 errors                              │
│ Impact: 40% of requests failing                                │
│ Severity: SEV 2                                                │
│                                                                 │
│ IC: @john                                                       │
│ Technical Lead: @jane                                          │
│                                                                 │
│ Status Page: Updated to Partial Outage                         │
│ Dashboard: [link]                                               │
└─────────────────────────────────────────────────────────────────┘
```

---

# 6. TOOLS

```
INCIDENT MANAGEMENT TOOLS:
┌─────────────────────────────────────────────────────────────────┐
│ ALERTING:                                                       │
│ - PagerDuty, OpsGenie, VictorOps                               │
│ - Azure Monitor, Prometheus Alertmanager                       │
│                                                                 │
│ COMMUNICATION:                                                  │
│ - Slack, Microsoft Teams                                       │
│ - Status pages (Statuspage.io, Atlassian)                      │
│                                                                 │
│ INCIDENT TRACKING:                                              │
│ - Jira, ServiceNow                                             │
│ - Incident.io, Rootly, FireHydrant                             │
│                                                                 │
│ POST-MORTEM:                                                    │
│ - Confluence, Notion                                           │
│ - Blameless, Jeli                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

# 7. METRICS & IMPROVEMENT

```
KEY METRICS TO TRACK:
┌─────────────────────────────────────────────────────────────────┐
│ MTTD:  Mean Time To Detect                                     │
│ MTTA:  Mean Time To Acknowledge                                │
│ MTTR:  Mean Time To Resolve                                    │
│ MTBF:  Mean Time Between Failures                              │
│                                                                 │
│ Incident Count:  By severity, by team, by service              │
│ On-Call Load:    Pages per shift, hours of incidents           │
│ Post-Mortem Rate: % of incidents with post-mortems            │
│ Action Item Completion: % of action items completed on time   │
└─────────────────────────────────────────────────────────────────┘

IMPROVEMENT AREAS:
- Reduce alert noise (consolidate, tune thresholds)
- Improve detection (better monitoring, synthetic tests)
- Automate remediation (self-healing)
- Better runbooks (from post-mortem learnings)
- Training (game days, chaos engineering)
```
