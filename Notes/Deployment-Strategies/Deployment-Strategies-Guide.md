# Deployment Strategies - Complete Guide
## Blue-Green, Canary, Rolling, Feature Flags

---

# 1. DEPLOYMENT FUNDAMENTALS

## Why Deployment Strategies Matter

```
RISKS OF BAD DEPLOYMENTS:
┌─────────────────────────────────────────────────────────────────┐
│ - Complete outage during deployment                            │
│ - Bugs reaching all users at once                              │
│ - Difficult or impossible rollback                             │
│ - Data corruption or loss                                      │
│ - Performance degradation                                       │
└─────────────────────────────────────────────────────────────────┘

GOALS OF GOOD DEPLOYMENT:
┌─────────────────────────────────────────────────────────────────┐
│ - Zero downtime                                                │
│ - Gradual rollout                                              │
│ - Easy rollback                                                │
│ - Minimal blast radius                                         │
│ - Observability                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

# 2. ROLLING UPDATE

## How It Works

```
ROLLING UPDATE:
Replace instances one at a time

Before:  [v1] [v1] [v1] [v1]

Step 1:  [v2] [v1] [v1] [v1]  ← First instance updated
Step 2:  [v2] [v2] [v1] [v1]  ← Second instance updated
Step 3:  [v2] [v2] [v2] [v1]  ← Third instance updated
Step 4:  [v2] [v2] [v2] [v2]  ← All updated

Traffic distributed across all instances throughout
```

## Kubernetes Implementation

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max extra pods during update
      maxUnavailable: 0  # Zero downtime
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:v2
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Update image
kubectl set image deployment/myapp myapp=myapp:v2

# Watch rollout
kubectl rollout status deployment/myapp

# Rollback
kubectl rollout undo deployment/myapp
```

## Pros/Cons

```
✓ PROS:
- Simple to implement
- No extra infrastructure
- Gradual rollout

✗ CONS:
- Mixed versions during rollout
- Can't test before full rollout
- Rollback takes time
```

---

# 3. BLUE-GREEN DEPLOYMENT

## How It Works

```
BLUE-GREEN DEPLOYMENT:
Two identical environments, switch traffic instantly

BEFORE (Blue active):
                    ┌──────────────────┐
  Traffic ─────────►│   Blue (v1)      │ ← Active
                    └──────────────────┘
                    ┌──────────────────┐
                    │   Green (idle)   │ ← Idle
                    └──────────────────┘

DEPLOY v2 TO GREEN:
                    ┌──────────────────┐
  Traffic ─────────►│   Blue (v1)      │ ← Still active
                    └──────────────────┘
                    ┌──────────────────┐
                    │   Green (v2)     │ ← Deploy & test
                    └──────────────────┘

SWITCH (Green active):
                    ┌──────────────────┐
                    │   Blue (v1)      │ ← Standby
                    └──────────────────┘
                    ┌──────────────────┐
  Traffic ─────────►│   Green (v2)     │ ← Active
                    └──────────────────┘
```

## Kubernetes Implementation

```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1

---
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2

---
# service.yaml - Switch by changing selector
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
    version: blue  # Change to 'green' to switch
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Switch traffic from blue to green
kubectl patch service myapp -p '{"spec":{"selector":{"version":"green"}}}'

# Rollback (switch back to blue)
kubectl patch service myapp -p '{"spec":{"selector":{"version":"blue"}}}'
```

## Pros/Cons

```
✓ PROS:
- Instant switch/rollback
- Test before going live
- No mixed versions

✗ CONS:
- Double infrastructure cost
- Database migrations tricky
- Stateful apps challenging
```

---

# 4. CANARY DEPLOYMENT

## How It Works

```
CANARY DEPLOYMENT:
Gradually shift traffic to new version

Step 1: 5% to canary
┌─────────────────┐         ┌───────────┐
│   Stable (v1)   │◄───95%──│  Traffic  │
└─────────────────┘         │           │
┌─────────────────┐         │           │
│   Canary (v2)   │◄────5%──│           │
└─────────────────┘         └───────────┘

Step 2: 25% to canary (if metrics good)
┌─────────────────┐         ┌───────────┐
│   Stable (v1)   │◄───75%──│  Traffic  │
└─────────────────┘         │           │
┌─────────────────┐         │           │
│   Canary (v2)   │◄───25%──│           │
└─────────────────┘         └───────────┘

Step 3: 100% to canary (promote)
┌─────────────────┐         ┌───────────┐
│   Stable (v2)   │◄──100%──│  Traffic  │
└─────────────────┘         └───────────┘
```

## Kubernetes with Istio

```yaml
# VirtualService for traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 95
    - destination:
        host: myapp
        subset: canary
      weight: 5

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: stable
    labels:
      version: v1
  - name: canary
    labels:
      version: v2
```

## Flagger (Automated Canary)

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  progressDeadlineSeconds: 60
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
    - name: request-duration
      thresholdRange:
        max: 500
```

## Pros/Cons

```
✓ PROS:
- Gradual rollout
- Real user testing
- Automatic rollback on metrics
- Minimal blast radius

✗ CONS:
- More complex setup
- Need good metrics
- Mixed versions in production
```

---

# 5. FEATURE FLAGS

## What Are Feature Flags?

```
FEATURE FLAG = Toggle feature on/off without deployment

CODE EXAMPLE:
if (featureFlags.isEnabled('new-checkout')) {
    showNewCheckout();
} else {
    showOldCheckout();
}

USE CASES:
┌─────────────────────────────────────────────────────────────────┐
│ Release toggles:   Deploy code, enable later                  │
│ Experiment toggles: A/B testing                                │
│ Ops toggles:       Kill switch for features                    │
│ Permission toggles: Features for specific users                │
└─────────────────────────────────────────────────────────────────┘
```

## Implementation

```javascript
// LaunchDarkly example
const LaunchDarkly = require('launchdarkly-node-server-sdk');
const client = LaunchDarkly.init('sdk-key');

async function handleRequest(user, req, res) {
  const showNewFeature = await client.variation('new-feature', user, false);
  
  if (showNewFeature) {
    return renderNewFeature(req, res);
  } else {
    return renderOldFeature(req, res);
  }
}

// User targeting
const user = {
  key: 'user-123',
  email: 'user@example.com',
  custom: {
    plan: 'premium',
    country: 'US',
  },
};
```

## Rollout Strategies

```
PROGRESSIVE ROLLOUT:
┌─────────────────────────────────────────────────────────────────┐
│ Day 1: Enable for internal team (1%)                          │
│ Day 2: Enable for beta users (5%)                             │
│ Day 3: Enable for 10% of users                                │
│ Day 4: Enable for 50% of users                                │
│ Day 5: Enable for 100% of users                               │
└─────────────────────────────────────────────────────────────────┘

TARGETING:
- By user ID (specific users)
- By percentage (random sample)
- By attribute (country, plan, etc.)
- By environment (prod, staging, etc.)
```

---

# 6. A/B TESTING

## How It Works

```
A/B TESTING:
Compare two versions with real users

┌─────────────────────────────────────────────────────────────────┐
│                        Users                                    │
│                          │                                      │
│              ┌───────────┴───────────┐                          │
│              ▼                       ▼                          │
│     ┌───────────────┐       ┌───────────────┐                  │
│     │  Variant A    │       │  Variant B    │                  │
│     │  (Control)    │       │  (Treatment)  │                  │
│     └───────┬───────┘       └───────┬───────┘                  │
│             │                       │                           │
│         Measure:                Measure:                        │
│         - Conversion            - Conversion                    │
│         - Revenue               - Revenue                       │
│         - Engagement            - Engagement                    │
│                                                                 │
│     Compare metrics to determine winner                         │
└─────────────────────────────────────────────────────────────────┘
```

---

# 7. DATABASE MIGRATIONS

## Safe Migration Strategies

```
EXPAND/CONTRACT PATTERN:

Phase 1: EXPAND (Add new)
- Add new column (nullable)
- Deploy app that writes to both old and new
- Backfill data

Phase 2: MIGRATE (Switch)
- Deploy app that reads from new
- Verify data consistency

Phase 3: CONTRACT (Remove old)
- Deploy app that only uses new
- Remove old column

EXAMPLE:
┌─────────────────────────────────────────────────────────────────┐
│ Goal: Rename 'email' column to 'email_address'                 │
│                                                                 │
│ Phase 1: ALTER TABLE users ADD email_address VARCHAR(255);     │
│          Deploy: Write to both email and email_address         │
│          Backfill: UPDATE users SET email_address = email;     │
│                                                                 │
│ Phase 2: Deploy: Read from email_address                       │
│          Verify: Compare email and email_address               │
│                                                                 │
│ Phase 3: Deploy: Stop using email column                       │
│          ALTER TABLE users DROP COLUMN email;                  │
└─────────────────────────────────────────────────────────────────┘
```

---

# 8. ROLLBACK STRATEGIES

```
ROLLBACK OPTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ INSTANT ROLLBACK:                                              │
│ - Blue-Green: Switch back to blue                              │
│ - Feature flags: Disable flag                                  │
│ - Canary: Shift traffic back                                   │
│                                                                 │
│ DEPLOYMENT ROLLBACK:                                            │
│ - kubectl rollout undo deployment/myapp                        │
│ - Redeploy previous version                                    │
│                                                                 │
│ DATABASE ROLLBACK:                                              │
│ - Forward-compatible migrations only                           │
│ - Avoid breaking changes                                       │
│ - Use versioned APIs                                           │
└─────────────────────────────────────────────────────────────────┘

ROLLBACK AUTOMATION:
- Automatic rollback on health check failure
- Automatic rollback on error rate threshold
- Automatic rollback on latency threshold
```

---

# 9. COMPARISON TABLE

```
┌──────────────┬──────────┬───────────┬────────────┬──────────────┐
│ Strategy     │ Downtime │ Rollback  │ Complexity │ Cost         │
├──────────────┼──────────┼───────────┼────────────┼──────────────┤
│ Recreate     │ Yes      │ Slow      │ Low        │ Low          │
│ Rolling      │ No       │ Medium    │ Low        │ Low          │
│ Blue-Green   │ No       │ Instant   │ Medium     │ High (2x)    │
│ Canary       │ No       │ Fast      │ High       │ Medium       │
│ Feature Flag │ No       │ Instant   │ Medium     │ Low          │
└──────────────┴──────────┴───────────┴────────────┴──────────────┘
```

---

# 10. BEST PRACTICES

```
DEPLOYMENT BEST PRACTICES:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Always have a rollback plan                                  │
│ ✓ Use health checks and readiness probes                      │
│ ✓ Implement progressive rollouts                               │
│ ✓ Monitor metrics during deployment                            │
│ ✓ Automate everything                                          │
│ ✓ Make deployments boring (frequent, small)                    │
│ ✓ Feature flags for risky changes                              │
│ ✓ Test in staging first                                        │
│ ✓ Deploy during low-traffic periods initially                  │
│ ✓ Have runbooks for deployment issues                          │
└─────────────────────────────────────────────────────────────────┘
```
