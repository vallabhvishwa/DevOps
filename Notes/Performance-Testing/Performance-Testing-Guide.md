# Performance & Load Testing - Complete Guide
## k6, JMeter, Load Testing Strategies, Analysis

---

# 1. PERFORMANCE TESTING TYPES

```
TESTING TYPES:
┌─────────────────────────────────────────────────────────────────┐
│ LOAD TESTING:                                                   │
│ - Test with expected load                                      │
│ - Verify SLAs are met                                          │
│ - Example: 1000 concurrent users for 30 minutes                │
│                                                                 │
│ STRESS TESTING:                                                 │
│ - Test beyond expected load                                    │
│ - Find breaking point                                          │
│ - Example: Increase users until errors occur                   │
│                                                                 │
│ SPIKE TESTING:                                                  │
│ - Sudden increase in load                                      │
│ - Test autoscaling, recovery                                   │
│ - Example: 100 → 10000 users instantly                         │
│                                                                 │
│ SOAK TESTING:                                                   │
│ - Extended duration testing                                    │
│ - Find memory leaks, resource exhaustion                       │
│ - Example: Normal load for 24 hours                            │
│                                                                 │
│ BASELINE TESTING:                                               │
│ - Establish performance benchmarks                             │
│ - Compare before/after changes                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# 2. KEY METRICS

```
PERFORMANCE METRICS:
┌─────────────────────────────────────────────────────────────────┐
│ RESPONSE TIME:                                                  │
│ - Average, Median (P50)                                        │
│ - P95, P99 (95th/99th percentile)                              │
│ - Maximum                                                       │
│                                                                 │
│ THROUGHPUT:                                                     │
│ - Requests per second (RPS)                                    │
│ - Transactions per second (TPS)                                │
│                                                                 │
│ ERROR RATE:                                                     │
│ - % of failed requests                                         │
│ - Types of errors (4xx, 5xx, timeouts)                         │
│                                                                 │
│ RESOURCE UTILIZATION:                                           │
│ - CPU, Memory usage                                            │
│ - Network I/O, Disk I/O                                        │
│ - Connection pool usage                                        │
│                                                                 │
│ CONCURRENCY:                                                    │
│ - Virtual users (VUs)                                          │
│ - Active connections                                           │
└─────────────────────────────────────────────────────────────────┘

WHY PERCENTILES MATTER:
Average: 200ms ← Looks good!

But reality:
P50: 100ms  (50% of requests)
P90: 300ms  (90% of requests)
P99: 5000ms (99% of requests) ← 1% of users wait 5 seconds!

Always measure P95/P99, not just average.
```

---

# 3. k6 (RECOMMENDED TOOL)

## Why k6?

```
k6 ADVANTAGES:
- JavaScript scripting
- Developer-friendly
- Git-friendly (tests as code)
- Built-in metrics and thresholds
- CI/CD integration
- Low resource usage
```

## Basic Script

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 100,           // Virtual users
  duration: '5m',     // Test duration
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],    // <1% errors
  },
};

export default function () {
  const res = http.get('https://api.example.com/users');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);  // Think time between requests
}
```

## Stages (Ramp Up/Down)

```javascript
export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 VUs
    { duration: '5m', target: 100 },   // Stay at 100 VUs
    { duration: '2m', target: 200 },   // Ramp up to 200 VUs
    { duration: '5m', target: 200 },   // Stay at 200 VUs
    { duration: '2m', target: 0 },     // Ramp down to 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
  },
};
```

## Advanced Scenarios

```javascript
import http from 'k6/http';
import { check, group, sleep } from 'k6';

export const options = {
  scenarios: {
    // Constant load
    constant_load: {
      executor: 'constant-vus',
      vus: 50,
      duration: '10m',
    },
    // Ramping load
    ramping_load: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '5m', target: 100 },
        { duration: '10m', target: 100 },
        { duration: '5m', target: 0 },
      ],
    },
    // Constant request rate
    constant_rate: {
      executor: 'constant-arrival-rate',
      rate: 100,           // 100 RPS
      timeUnit: '1s',
      duration: '10m',
      preAllocatedVUs: 50,
    },
  },
};

export default function () {
  group('API Flow', function () {
    // Login
    const loginRes = http.post('https://api.example.com/login', {
      username: 'testuser',
      password: 'testpass',
    });
    check(loginRes, { 'login successful': (r) => r.status === 200 });
    
    const token = loginRes.json('token');
    
    // Get data
    const dataRes = http.get('https://api.example.com/data', {
      headers: { Authorization: `Bearer ${token}` },
    });
    check(dataRes, { 'got data': (r) => r.status === 200 });
  });
  
  sleep(1);
}
```

## Running k6

```bash
# Basic run
k6 run load-test.js

# With more VUs and duration
k6 run --vus 200 --duration 10m load-test.js

# Output to JSON
k6 run --out json=results.json load-test.js

# Cloud run (k6 Cloud)
k6 cloud load-test.js
```

---

# 4. JMETER

## Basic Test Plan

```
JMETER STRUCTURE:
┌─────────────────────────────────────────────────────────────────┐
│ Test Plan                                                       │
│ ├── Thread Group (Virtual Users)                               │
│ │   ├── HTTP Request Sampler                                  │
│ │   ├── Response Assertion                                    │
│ │   └── Timer (Think Time)                                    │
│ ├── Listeners (Results)                                        │
│ │   ├── Summary Report                                        │
│ │   └── Response Time Graph                                   │
│ └── Config Elements                                            │
│     ├── HTTP Header Manager                                    │
│     └── CSV Data Set Config                                    │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Run test
jmeter -n -t test-plan.jmx -l results.jtl

# Generate HTML report
jmeter -g results.jtl -o report-directory

# With properties
jmeter -n -t test.jmx -Jusers=100 -Jduration=300 -l results.jtl
```

---

# 5. TESTING STRATEGIES

## Test Design

```
TEST DESIGN STEPS:
┌─────────────────────────────────────────────────────────────────┐
│ 1. DEFINE OBJECTIVES                                           │
│    - What are we testing? (API, web app, database)             │
│    - What are the success criteria?                            │
│    - What are the SLAs?                                        │
│                                                                 │
│ 2. IDENTIFY SCENARIOS                                          │
│    - Most common user flows                                    │
│    - Critical business transactions                            │
│    - Peak load patterns                                        │
│                                                                 │
│ 3. DETERMINE LOAD MODEL                                        │
│    - Expected concurrent users                                 │
│    - Requests per second                                       │
│    - Think times                                               │
│    - Session duration                                          │
│                                                                 │
│ 4. SET THRESHOLDS                                              │
│    - Response time (P95 < 500ms)                               │
│    - Error rate (< 1%)                                         │
│    - Throughput (> 1000 RPS)                                   │
│                                                                 │
│ 5. PREPARE ENVIRONMENT                                         │
│    - Test environment similar to production                    │
│    - Test data                                                 │
│    - Monitoring in place                                       │
└─────────────────────────────────────────────────────────────────┘
```

## Load Calculation

```
CALCULATING LOAD:

Given:
- 10,000 daily active users
- Average session: 10 minutes
- Peak hour: 20% of daily traffic

Concurrent users at peak:
= (Daily users × Peak %) × (Session duration / 60)
= (10,000 × 0.20) × (10 / 60)
= 2,000 × 0.167
= ~333 concurrent users

Add buffer for growth:
= 333 × 1.5 (50% buffer)
= ~500 concurrent users for testing
```

---

# 6. CI/CD INTEGRATION

```yaml
# GitHub Actions example
name: Performance Tests
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  k6-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install k6
        run: |
          sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update
          sudo apt-get install k6
      
      - name: Run load test
        run: k6 run --out json=results.json tests/load-test.js
      
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: k6-results
          path: results.json
```

---

# 7. ANALYSIS & REPORTING

```
ANALYSIS CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│ ☐ Did we meet response time SLAs?                             │
│ ☐ What was the error rate?                                    │
│ ☐ At what load did performance degrade?                       │
│ ☐ What was the bottleneck? (CPU, memory, DB, network)         │
│ ☐ Did autoscaling work as expected?                           │
│ ☐ Were there any memory leaks?                                │
│ ☐ How does it compare to baseline?                            │
└─────────────────────────────────────────────────────────────────┘

COMMON BOTTLENECKS:
┌─────────────────────────────────────────────────────────────────┐
│ Application:    CPU-bound code, inefficient algorithms         │
│ Database:       Slow queries, missing indexes, connection pool │
│ Network:        Bandwidth, latency, connection limits          │
│ Infrastructure: Undersized instances, disk I/O                 │
│ External:       Third-party API rate limits                    │
└─────────────────────────────────────────────────────────────────┘
```

---

# 8. BEST PRACTICES

```
PERFORMANCE TESTING BEST PRACTICES:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Test early and often (shift left)                           │
│ ✓ Use realistic test data                                     │
│ ✓ Include think times                                         │
│ ✓ Monitor backend during tests                                │
│ ✓ Test from multiple locations                                │
│ ✓ Establish baselines                                         │
│ ✓ Version control your tests                                  │
│ ✓ Integrate into CI/CD                                        │
│ ✓ Test in production-like environment                         │
│ ✓ Document and share results                                  │
└─────────────────────────────────────────────────────────────────┘
```
