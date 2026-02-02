# Jenkins Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

---

# SECTION 1: BUILD FAILURES

---

## 1.1 BUILD WON'T START

### LEVEL 1 - DIRECT: No Executor Available

**Scenario**: Build stuck in queue.

```
Build Queue:
myproject #42 - Waiting for next available executor
```

**Cause is VISIBLE**: No free executors.

**Solution**:
```bash
# Check executors
# Manage Jenkins > Nodes
# See how many executors are available

# Either:
# - Wait for current builds to finish
# - Add more executors
# - Add more agents
```

---

### LEVEL 1 - DIRECT: Agent Offline

**Scenario**: Build waiting for specific agent.

```
Waiting for next available executor on 'build-agent-1'
(Offline)
```

**Cause is VISIBLE**: Agent is offline.

**Solution**:
```bash
# Check why agent is offline
# Manage Jenkins > Nodes > build-agent-1

# Reconnect agent
# Or fix the underlying issue (SSH, network, etc.)
```

---

### LEVEL 1 - DIRECT: Label Mismatch

**Scenario**: Build waiting, agents available but unused.

```
Waiting for next available executor on 'docker-agent'
# But agents are idle!
```

**Cause is VISIBLE**: No agent has the 'docker-agent' label.

**Solution**:
```bash
# Check agent labels
# Manage Jenkins > Nodes > <agent> > Configure

# Either:
# - Add label to existing agent
# - Change pipeline to use existing label
```

---

## 1.2 BUILD FAILS

### LEVEL 1 - DIRECT: Command Not Found

**Scenario**: Build step fails with command not found.

```bash
+ mvn clean install
/bin/sh: mvn: command not found
```

**Cause is VISIBLE**: Maven not installed or not in PATH.

**Solution**:
```groovy
// Option 1: Use tools block
pipeline {
  tools {
    maven 'Maven-3.8'  // Configured in Global Tool Configuration
  }
}

// Option 2: Use full path
sh '/usr/local/maven/bin/mvn clean install'
```

---

### LEVEL 1 - DIRECT: Permission Denied

**Scenario**: Can't execute script.

```bash
+ ./build.sh
/bin/sh: ./build.sh: Permission denied
```

**Cause is VISIBLE**: Script not executable.

**Solution**:
```groovy
sh 'chmod +x ./build.sh && ./build.sh'
```

---

### LEVEL 1 - DIRECT: Test Failed

**Scenario**: Build fails due to test failure.

```bash
Tests run: 50, Failures: 2, Errors: 0
BUILD FAILURE
```

**Cause is VISIBLE**: 2 tests failed.

**Solution**:
```bash
# Check test report for which tests failed
# Fix the tests or code

# To allow build to continue despite test failures:
sh 'mvn test || true'  # Not recommended for production
```

---

### LEVEL 2 - INTERMEDIATE: Works Locally, Fails in Jenkins

**Scenario**: Developer says "it works on my machine".

```bash
# Local: mvn clean install - SUCCESS
# Jenkins: mvn clean install - FAILURE
```

**Investigation**:
```groovy
// Debug environment
stage('Debug') {
  steps {
    sh '''
      echo "=== Environment ==="
      env | sort
      
      echo "=== Java Version ==="
      java -version
      
      echo "=== Maven Version ==="
      mvn -version
      
      echo "=== Workspace ==="
      pwd
      ls -la
    '''
  }
}
```

**Common Differences**:
- Different tool versions
- Missing environment variables
- Different user/permissions
- Workspace state from previous builds

---

### LEVEL 3 - COMPLEX: Flaky Tests (Sometimes Fail)

**Scenario**: Build fails randomly, same test sometimes passes.

**Hidden Causes**:
```
FLAKY TEST CAUSES:
┌─────────────────────────────────────────────────────────────────┐
│  1. Race conditions                                             │
│  2. Order dependency between tests                              │
│  3. External service dependency (API, DB)                       │
│  4. Time-based tests                                            │
│  5. Resource constraints (memory, CPU)                          │
└─────────────────────────────────────────────────────────────────┘
```

**Solution**:
```groovy
// Retry flaky stages
stage('Test') {
  steps {
    retry(3) {
      sh 'mvn test'
    }
  }
}

// Or quarantine flaky tests
// Mark with @Ignore temporarily and investigate
```

---

## 1.3 PIPELINE ISSUES

### LEVEL 1 - DIRECT: Pipeline Syntax Error

**Scenario**: Pipeline fails to parse.

```
WorkflowScript: Expected a step
```

**Cause is VISIBLE**: Jenkinsfile syntax error.

**Solution**:
```groovy
// Use the Pipeline Syntax generator
// Pipeline Syntax > Snippet Generator

// Common issues:
// - Missing 'steps' block
// - Wrong indentation
// - Missing quotes
```

---

### LEVEL 1 - DIRECT: Missing Credentials

**Scenario**: Can't access credentials.

```
CredentialNotFoundException: No such credential: github-token
```

**Cause is VISIBLE**: Credential doesn't exist.

**Solution**:
```bash
# Create credential
# Manage Jenkins > Manage Credentials > Add Credentials

# Make sure scope is correct (Global vs Folder)
```

---

### LEVEL 2 - INTERMEDIATE: Pipeline Hangs

**Scenario**: Pipeline stuck, never completes.

**Investigation**:
```groovy
// Common causes:
// 1. Waiting for input
// 2. Script waiting for user input
// 3. Infinite loop
// 4. Network timeout

// Add timeout
timeout(time: 30, unit: 'MINUTES') {
  sh 'long-running-script.sh'
}
```

**Solution**:
```groovy
// Make scripts non-interactive
sh 'echo "y" | command-that-prompts'

// Use proper timeout
timeout(time: 1, unit: 'HOURS') {
  stage('Build') {
    // ...
  }
}
```

---

# SECTION 2: KUBERNETES AGENTS

---

## 2.1 AGENT WON'T START

### LEVEL 1 - DIRECT: Pod Not Created

**Scenario**: Pipeline waits, no pod created.

**Investigation**:
```bash
# Check Jenkins logs
kubectl logs <jenkins-controller> | grep -i "agent\|pod"

# Check Kubernetes cloud config
# Manage Jenkins > Manage Nodes and Clouds > Configure Clouds
```

**Common Issues**:
- Wrong Kubernetes URL
- Missing credentials
- Invalid pod template

---

### LEVEL 1 - DIRECT: Pod Pending

**Scenario**: Agent pod created but pending.

```bash
$ kubectl get pods -n jenkins
NAME                         STATUS    
jenkins-agent-xyz            Pending
```

**Solution**:
```bash
# Check pod events
kubectl describe pod jenkins-agent-xyz -n jenkins

# Common issues:
# - Resource constraints
# - Node selector mismatch
# - PVC not binding
```

---

### LEVEL 2 - INTERMEDIATE: JNLP Connection Failed

**Scenario**: Pod starts but agent never connects.

```bash
# Pod is Running
# But Jenkins shows agent offline
```

**Investigation**:
```bash
# Check JNLP container logs
kubectl logs jenkins-agent-xyz -c jnlp -n jenkins

# Common issues:
# - Jenkins URL wrong
# - Firewall blocking
# - DNS not resolving
```

**Solution**:
```yaml
# Ensure JENKINS_URL is correct
env:
  - name: JENKINS_URL
    value: "http://jenkins.jenkins.svc.cluster.local:8080"
```

---

## 2.2 BUILD ISSUES ON K8S AGENTS

### LEVEL 1 - DIRECT: Container Not Found

**Scenario**: Can't use specified container.

```
ERROR: No such container: build
```

**Cause is VISIBLE**: Container name mismatch.

**Solution**:
```groovy
// Check container names in pod template match what you use
podTemplate(containers: [
  containerTemplate(name: 'maven', image: 'maven:3.8')
]) {
  container('maven') {  // Must match name above
    sh 'mvn --version'
  }
}
```

---

### LEVEL 1 - DIRECT: Tool Not in Container

**Scenario**: Command not found in container.

```bash
+ docker build .
docker: not found
```

**Cause is VISIBLE**: Docker not installed in that container.

**Solution**:
```groovy
// Use container with Docker
podTemplate(containers: [
  containerTemplate(name: 'docker', image: 'docker:dind')
]) {
  container('docker') {
    sh 'docker build .'
  }
}
```

---

# SECTION 3: PERFORMANCE

---

## 3.1 JENKINS SLOW

### LEVEL 1 - DIRECT: High Memory Usage

**Scenario**: Jenkins UI slow, memory high.

```bash
$ top
PID   %MEM   COMMAND
1234  80.0   java  # Jenkins using 80% RAM
```

**Cause is VISIBLE**: Jenkins using too much memory.

**Solution**:
```bash
# Increase heap
JAVA_OPTS="-Xmx4g -Xms2g"

# Check for plugins consuming memory
# Manage Jenkins > System Information
```

---

### LEVEL 2 - INTERMEDIATE: Slow Over Time

**Scenario**: Jenkins gets slower over weeks/months.

**Investigation**:
- Too many builds stored
- Large job configurations
- Too many plugins

**Solution**:
```groovy
// Add build discarder
options {
  buildDiscarder(logRotator(numToKeepStr: '10'))
}

// Clean workspaces
post {
  always {
    cleanWs()
  }
}
```

---

# QUICK REFERENCE

## Common Pipeline Issues

```groovy
// PROBLEM: Credential exposed in logs
sh "curl -u $USER:$PASS https://..."  // Shows in logs!

// SOLUTION: Use single quotes (Groovy doesn't interpolate)
withCredentials([...]) {
  sh 'curl -u $USER:$PASS https://...'  // $PASS masked
}

// PROBLEM: Shell script errors don't fail build
sh '''
  command1
  failing-command
  command2  # Still runs!
'''

// SOLUTION: Use set -e
sh '''
  set -e
  command1
  failing-command
  command2  # Won't run
'''

// PROBLEM: Pipeline timeout
// No timeout = can run forever

// SOLUTION: Add timeout
options {
  timeout(time: 1, unit: 'HOURS')
}
```

## Diagnostic Locations

```
Jenkins Logs: Manage Jenkins > System Log
Build Log: Job > Build > Console Output
Agent Logs: Manage Jenkins > Nodes > <agent> > Log
Script Console: Manage Jenkins > Script Console
```

## Common Errors → Solutions

```
"No valid crumb"               → CSRF issue, enable crumb
"Agent went offline"           → Check agent connectivity
"Waiting for executor"         → Add executors or agents
"Disk space is too low"        → Clean old builds
"Script not approved"          → Approve in Script Approval
```
