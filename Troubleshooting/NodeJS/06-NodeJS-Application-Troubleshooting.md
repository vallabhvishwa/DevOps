# Node.js Application Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

> **Learning reference:** [Observability & Node.js](../../Notes/Observability/DevOps-07-Observability-Apps.md)

---

# SECTION 1: STARTUP ISSUES

---

## 1.1 APPLICATION WON'T START

### LEVEL 1 - DIRECT: Module Not Found

**Scenario**: App fails with missing module.

```bash
$ node app.js
Error: Cannot find module 'express'
```

**Cause is VISIBLE**: Express package not installed.

**Solution**:
```bash
# Install dependencies
npm install

# Or install specific package
npm install express
```

---

### LEVEL 1 - DIRECT: Syntax Error

**Scenario**: JavaScript syntax error.

```bash
$ node app.js
SyntaxError: Unexpected token '}'
    at app.js:15:1
```

**Cause is VISIBLE**: Syntax error at line 15.

**Solution**:
```bash
# Fix the syntax error at line 15
# Check for missing bracket, semicolon, etc.
```

---

### LEVEL 1 - DIRECT: Port In Use

**Scenario**: Port already taken.

```bash
$ node app.js
Error: listen EADDRINUSE: address already in use :::3000
```

**Cause is VISIBLE**: Port 3000 is already in use.

**Solution**:
```bash
# Find what's using the port
lsof -i :3000

# Kill it or use different port
PORT=3001 node app.js
```

---

### LEVEL 1 - DIRECT: File Not Found

**Scenario**: Required file doesn't exist.

```bash
$ node app.js
Error: ENOENT: no such file or directory, open './config.json'
```

**Cause is VISIBLE**: config.json doesn't exist.

**Solution**:
```bash
# Create the file
echo '{}' > config.json

# Or fix the path in code
```

---

### LEVEL 2 - INTERMEDIATE: Wrong Node Version

**Scenario**: App uses modern syntax, old Node.

```bash
$ node app.js
SyntaxError: Unexpected token '...'  # Spread operator
```

**Investigation**:
```bash
# Check Node version
node --version
# v8.0.0 (too old!)

# Check required version in package.json
cat package.json | grep "engines"
```

**Solution**:
```bash
# Upgrade Node
nvm install 18
nvm use 18
```

---

## 1.2 DEPENDENCY ISSUES

### LEVEL 1 - DIRECT: npm install Failed

**Scenario**: npm install errors.

```bash
$ npm install
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
```

**Cause is VISIBLE**: Dependency conflict.

**Solution**:
```bash
# Force install (careful!)
npm install --legacy-peer-deps

# Or update conflicting dependencies
npm update
```

---

### LEVEL 1 - DIRECT: Native Module Build Failed

**Scenario**: Package with native code fails.

```bash
$ npm install
npm ERR! node-pre-gyp ERR! build error
npm ERR! make failed with exit code 2
```

**Cause is VISIBLE**: Missing build tools.

**Solution**:
```bash
# Install build tools
# Ubuntu/Debian:
apt-get install build-essential

# Alpine:
apk add python3 make g++

# macOS:
xcode-select --install
```

---

# SECTION 2: MEMORY ISSUES

---

## 2.1 OUT OF MEMORY

### LEVEL 1 - DIRECT: Heap Out of Memory

**Scenario**: App crashes with heap OOM.

```bash
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

**Cause is VISIBLE**: V8 heap is full.

**Solution**:
```bash
# Increase heap size
node --max-old-space-size=4096 app.js

# Or set via environment
NODE_OPTIONS="--max-old-space-size=4096" node app.js
```

---

### LEVEL 2 - INTERMEDIATE: Memory Grows Over Time

**Scenario**: Memory usage increases gradually.

```bash
# After 1 hour: 200MB
# After 4 hours: 800MB
# After 8 hours: OOM
```

**Investigation**:
```bash
# Check memory usage
node -e "console.log(process.memoryUsage())"

# Take heap snapshot
# Add to app:
const v8 = require('v8');
const fs = require('fs');
fs.writeFileSync('heap.heapsnapshot', v8.writeHeapSnapshot());

# Or use Chrome DevTools
node --inspect app.js
# Open chrome://inspect
```

**Common Leak Causes**:
- Event listeners not removed
- Growing arrays/objects
- Closures holding references
- Timers not cleared

---

### LEVEL 3 - COMPLEX: Memory Leak in Dependencies

**Scenario**: Your code looks fine but memory still leaks.

**Deep Investigation**:
```bash
# Profile with clinic.js
npx clinic doctor -- node app.js

# Check for known issues in dependencies
npm audit

# Update dependencies
npm update
```

---

# SECTION 3: PERFORMANCE ISSUES

---

## 3.1 HIGH CPU / SLOW RESPONSE

### LEVEL 1 - DIRECT: Blocking Operation in Request

**Scenario**: One request blocks all others.

```javascript
// Bad: Blocking operation
app.get('/compute', (req, res) => {
  const result = heavyComputation();  // BLOCKS!
  res.json(result);
});
```

**Cause is VISIBLE**: Synchronous heavy computation.

**Solution**:
```javascript
// Use worker threads for CPU-intensive work
const { Worker } = require('worker_threads');

app.get('/compute', (req, res) => {
  const worker = new Worker('./worker.js');
  worker.on('message', result => res.json(result));
});
```

---

### LEVEL 1 - DIRECT: Sync File Operations

**Scenario**: App slow, using sync file methods.

```javascript
// Bad: Blocking I/O
const data = fs.readFileSync('large-file.txt');
```

**Cause is VISIBLE**: Using `readFileSync` (sync = blocking).

**Solution**:
```javascript
// Use async version
const data = await fs.promises.readFile('large-file.txt');

// Or callback version
fs.readFile('large-file.txt', (err, data) => {...});
```

---

### LEVEL 2 - INTERMEDIATE: Event Loop Lag

**Scenario**: Responses slow but no obvious sync code.

**Investigation**:
```javascript
// Measure event loop lag
let lastCheck = Date.now();
setInterval(() => {
  const now = Date.now();
  const lag = now - lastCheck - 100;  // Should be ~0
  if (lag > 10) console.log('Event loop lag:', lag, 'ms');
  lastCheck = now;
}, 100);
```

**Common Causes**:
- JSON parsing large objects
- Array operations on large arrays
- Regular expressions on long strings
- Crypto operations

---

### LEVEL 3 - COMPLEX: Application Hangs Randomly

**Scenario**: App stops responding, then recovers.

**Hidden Causes**:
```
RANDOM HANGS:
┌─────────────────────────────────────────────────────────────────┐
│  1. GC pause (can be seconds for large heap)                    │
│  2. DNS resolution blocking                                     │
│  3. Connection pool exhausted                                   │
│  4. Promise never resolving                                     │
│  5. Infinite loop in callback                                   │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check for GC issues
node --trace-gc app.js

# Check for unhandled promises
node --unhandled-rejections=strict app.js

# Profile with 0x
npx 0x app.js
```

---

## 3.2 CONNECTION ISSUES

### LEVEL 1 - DIRECT: Database Connection Failed

**Scenario**: Can't connect to database.

```bash
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Cause is VISIBLE**: PostgreSQL not running or wrong host.

**Solution**:
```bash
# Check if database is running
systemctl status postgresql

# Check connection settings
# Host, port, credentials
```

---

### LEVEL 2 - INTERMEDIATE: Connection Pool Exhausted

**Scenario**: "Too many connections" or timeout getting connection.

```bash
Error: timeout exceeded when trying to connect
# Or
Error: too many connections
```

**Investigation**:
```javascript
// Check pool settings
const pool = new Pool({
  max: 10,  // Only 10 connections allowed
  connectionTimeoutMillis: 5000
});

// Are connections being released?
pool.query('SELECT 1')
  .finally(() => /* Connection returned to pool */);
```

**Solution**:
```javascript
// Increase pool size
const pool = new Pool({ max: 50 });

// Make sure to handle errors
try {
  const result = await pool.query('...');
} catch (err) {
  console.error(err);
} // Connection released even on error
```

---

# SECTION 4: CONTAINER ISSUES

---

## 4.1 NODE IN CONTAINERS

### LEVEL 1 - DIRECT: Container Exits Immediately

**Scenario**: Container starts and exits right away.

**Cause is VISIBLE**: npm start uses foreground process.

**Solution**:
```dockerfile
# Correct: Keep process in foreground
CMD ["node", "server.js"]

# Wrong: Shell might exit
CMD npm start  # Uses shell form
```

---

### LEVEL 1 - DIRECT: Signals Not Handled

**Scenario**: Container takes 30s to stop.

**Cause is VISIBLE**: App not handling SIGTERM.

**Solution**:
```javascript
// Handle shutdown gracefully
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down');
  
  // Close server
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // Close database connections
  await pool.end();
  
  process.exit(0);
});
```

---

### LEVEL 2 - INTERMEDIATE: V8 Doesn't Respect Container Limits

**Scenario**: Container OOMKilled even with memory available.

```bash
# Container has 512MB limit
# V8 default heap ~1.5GB
# Container gets OOMKilled
```

**Solution**:
```dockerfile
# Set heap size to match container
ENV NODE_OPTIONS="--max-old-space-size=384"

# Leave room for non-heap memory
# Container: 512MB
# Heap: 384MB
# Other: 128MB for stack, buffers, etc.
```

---

# QUICK REFERENCE

## Diagnostic Commands

```bash
# Memory
node -e "console.log(process.memoryUsage())"

# V8 heap stats
node -e "console.log(require('v8').getHeapStatistics())"

# Debug mode
node --inspect app.js
node --inspect-brk app.js  # Break on first line

# Trace warnings
node --trace-warnings app.js

# CPU profiling
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# Heap snapshot
node --heapsnapshot-signal=SIGUSR2 app.js
kill -USR2 <pid>  # Generates snapshot
```

## Common Errors → Solutions

```
"Cannot find module"           → npm install or check path
"Heap out of memory"           → --max-old-space-size
"EADDRINUSE"                   → Port conflict
"ECONNREFUSED"                 → Service not running
"ENOENT"                       → File not found
"ETIMEDOUT"                    → Network timeout
"Unhandled promise rejection"  → Add catch handler
"EMFILE too many open files"   → Increase ulimit -n
```

## Production Settings

```bash
# Environment
NODE_ENV=production

# Memory
NODE_OPTIONS="--max-old-space-size=4096"

# Cluster mode (use all CPUs)
# Use PM2 or cluster module

# Logging
NODE_DEBUG=http,net node app.js
```
