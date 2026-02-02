# DevOps Engineer's Complete Reference Guide
# Part 15: Python & Go for DevOps

---

## Table of Contents

1. [Why Learn Programming for DevOps](#1-why-learn-programming-for-devops)
2. [Python for DevOps](#2-python-for-devops)
3. [Go for DevOps](#3-go-for-devops)
4. [Common DevOps Automation Tasks](#4-common-devops-automation-tasks)
5. [Building CLI Tools](#5-building-cli-tools)
6. [Kubernetes Operators](#6-kubernetes-operators)
7. [Testing and Best Practices](#7-testing-and-best-practices)

---

## 1. Why Learn Programming for DevOps

```
Programming in DevOps:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  FAANG Expectations:                                            │
│  ├── Automate complex workflows                                 │
│  ├── Build internal tools                                       │
│  ├── Write Kubernetes operators                                 │
│  ├── Create custom monitoring solutions                         │
│  ├── Develop CI/CD plugins                                      │
│  └── Pass coding interviews (LeetCode easy/medium)              │
│                                                                 │
│  Python vs Go:                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Python                      │ Go                           │ │
│  │────────────────────────────────────────────────────────────│ │
│  │ Fast to write               │ Fast to execute              │ │
│  │ Great for scripting         │ Great for CLIs, operators    │ │
│  │ Rich ecosystem              │ Kubernetes native            │ │
│  │ AWS/Azure/GCP SDKs          │ Static binary (no deps)      │ │
│  │ Ansible, SaltStack          │ Terraform, K8s, Docker       │ │
│  │ Interview friendly          │ Cloud-native preferred       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                 │
│  Recommendation: Learn both!                                    │
│  ├── Python: Day-to-day automation, scripts, quick tools        │
│  └── Go: Production tools, operators, performance-critical      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Python for DevOps

### 2.1 Essential Python Concepts

```python
# Data Structures
# ─────────────────────────────────────────────────────────────────

# Lists
servers = ["web-1", "web-2", "web-3"]
servers.append("web-4")
active_servers = [s for s in servers if "web" in s]  # List comprehension

# Dictionaries
config = {
    "host": "localhost",
    "port": 8080,
    "debug": True
}
port = config.get("port", 80)  # Default value if key missing

# Sets (unique values)
deployed_versions = {"v1.0", "v1.1", "v1.0"}  # Only unique: {"v1.0", "v1.1"}

# ─────────────────────────────────────────────────────────────────
# Functions
# ─────────────────────────────────────────────────────────────────

def deploy_service(name: str, version: str, replicas: int = 1) -> bool:
    """Deploy a service to the cluster."""
    print(f"Deploying {name}:{version} with {replicas} replicas")
    return True

# Lambda functions
sort_by_name = lambda x: x["name"]
sorted_services = sorted(services, key=sort_by_name)

# ─────────────────────────────────────────────────────────────────
# Classes
# ─────────────────────────────────────────────────────────────────

class KubernetesClient:
    def __init__(self, config_path: str = None):
        self.config_path = config_path or "~/.kube/config"
        self._load_config()
    
    def _load_config(self):
        # Private method
        pass
    
    def list_pods(self, namespace: str = "default") -> list:
        """List all pods in a namespace."""
        # Implementation
        return []

# ─────────────────────────────────────────────────────────────────
# Error Handling
# ─────────────────────────────────────────────────────────────────

try:
    result = risky_operation()
except ConnectionError as e:
    logger.error(f"Connection failed: {e}")
    raise
except Exception as e:
    logger.exception("Unexpected error")
finally:
    cleanup()

# ─────────────────────────────────────────────────────────────────
# Context Managers (with statement)
# ─────────────────────────────────────────────────────────────────

# File handling
with open("config.yaml", "r") as f:
    config = yaml.safe_load(f)

# Custom context manager
from contextlib import contextmanager

@contextmanager
def timer(name: str):
    start = time.time()
    yield
    elapsed = time.time() - start
    print(f"{name} took {elapsed:.2f}s")

with timer("deployment"):
    deploy_application()
```

### 2.2 Python DevOps Libraries

```python
# ─────────────────────────────────────────────────────────────────
# Kubernetes Client
# ─────────────────────────────────────────────────────────────────
from kubernetes import client, config

# Load kubeconfig
config.load_kube_config()  # From ~/.kube/config
# config.load_incluster_config()  # From within cluster

v1 = client.CoreV1Api()
apps_v1 = client.AppsV1Api()

# List pods
pods = v1.list_namespaced_pod(namespace="default")
for pod in pods.items:
    print(f"{pod.metadata.name}: {pod.status.phase}")

# Create deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="myapp"),
    spec=client.V1DeploymentSpec(
        replicas=3,
        selector=client.V1LabelSelector(
            match_labels={"app": "myapp"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={"app": "myapp"}),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="myapp",
                        image="myapp:latest",
                        ports=[client.V1ContainerPort(container_port=8080)]
                    )
                ]
            )
        )
    )
)
apps_v1.create_namespaced_deployment(namespace="default", body=deployment)

# ─────────────────────────────────────────────────────────────────
# Azure SDK
# ─────────────────────────────────────────────────────────────────
from azure.identity import DefaultAzureCredential
from azure.mgmt.containerservice import ContainerServiceClient
from azure.mgmt.resource import ResourceManagementClient

credential = DefaultAzureCredential()
subscription_id = "your-subscription-id"

# Resource management
resource_client = ResourceManagementClient(credential, subscription_id)

# List resource groups
for rg in resource_client.resource_groups.list():
    print(rg.name)

# AKS management
aks_client = ContainerServiceClient(credential, subscription_id)
clusters = aks_client.managed_clusters.list()
for cluster in clusters:
    print(f"{cluster.name}: {cluster.kubernetes_version}")

# ─────────────────────────────────────────────────────────────────
# HTTP Requests
# ─────────────────────────────────────────────────────────────────
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# Session with retries
session = requests.Session()
retries = Retry(total=3, backoff_factor=0.5, status_forcelist=[500, 502, 503])
session.mount('https://', HTTPAdapter(max_retries=retries))

# API calls
response = session.get(
    "https://api.example.com/health",
    headers={"Authorization": f"Bearer {token}"},
    timeout=10
)
response.raise_for_status()
data = response.json()

# ─────────────────────────────────────────────────────────────────
# Async Operations
# ─────────────────────────────────────────────────────────────────
import asyncio
import aiohttp

async def check_health(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return {"url": url, "status": response.status}

async def check_all_services(urls: list) -> list:
    tasks = [check_health(url) for url in urls]
    return await asyncio.gather(*tasks)

# Run async
results = asyncio.run(check_all_services([
    "https://api1.example.com/health",
    "https://api2.example.com/health",
]))
```

### 2.3 Practical Python Scripts

```python
#!/usr/bin/env python3
"""
Kubernetes deployment automation script.
"""
import argparse
import logging
import sys
from kubernetes import client, config

logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')
logger = logging.getLogger(__name__)

def rollout_restart(namespace: str, deployment: str) -> bool:
    """Perform rolling restart of a deployment."""
    config.load_kube_config()
    apps_v1 = client.AppsV1Api()
    
    # Get current deployment
    try:
        dep = apps_v1.read_namespaced_deployment(deployment, namespace)
    except client.ApiException as e:
        logger.error(f"Deployment not found: {e}")
        return False
    
    # Trigger restart by updating annotation
    from datetime import datetime
    if dep.spec.template.metadata.annotations is None:
        dep.spec.template.metadata.annotations = {}
    
    dep.spec.template.metadata.annotations["kubectl.kubernetes.io/restartedAt"] = \
        datetime.utcnow().isoformat()
    
    try:
        apps_v1.patch_namespaced_deployment(deployment, namespace, dep)
        logger.info(f"Restarted deployment {deployment} in {namespace}")
        return True
    except client.ApiException as e:
        logger.error(f"Failed to restart: {e}")
        return False

def scale_deployment(namespace: str, deployment: str, replicas: int) -> bool:
    """Scale a deployment."""
    config.load_kube_config()
    apps_v1 = client.AppsV1Api()
    
    body = {"spec": {"replicas": replicas}}
    try:
        apps_v1.patch_namespaced_deployment_scale(deployment, namespace, body)
        logger.info(f"Scaled {deployment} to {replicas} replicas")
        return True
    except client.ApiException as e:
        logger.error(f"Failed to scale: {e}")
        return False

def main():
    parser = argparse.ArgumentParser(description="Kubernetes deployment tool")
    parser.add_argument("action", choices=["restart", "scale"])
    parser.add_argument("-n", "--namespace", default="default")
    parser.add_argument("-d", "--deployment", required=True)
    parser.add_argument("-r", "--replicas", type=int)
    
    args = parser.parse_args()
    
    if args.action == "restart":
        success = rollout_restart(args.namespace, args.deployment)
    elif args.action == "scale":
        if args.replicas is None:
            logger.error("--replicas required for scale action")
            sys.exit(1)
        success = scale_deployment(args.namespace, args.deployment, args.replicas)
    
    sys.exit(0 if success else 1)

if __name__ == "__main__":
    main()
```

---

## 3. Go for DevOps

### 3.1 Go Basics

```go
package main

import (
    "fmt"
    "log"
    "time"
)

// ─────────────────────────────────────────────────────────────────
// Variables and Types
// ─────────────────────────────────────────────────────────────────

func main() {
    // Variable declaration
    var name string = "myapp"
    version := "1.0.0"  // Short declaration
    
    // Arrays and Slices
    servers := []string{"web-1", "web-2", "web-3"}
    servers = append(servers, "web-4")
    
    // Maps
    config := map[string]interface{}{
        "host":  "localhost",
        "port":  8080,
        "debug": true,
    }
    
    // Structs
    type Deployment struct {
        Name      string
        Namespace string
        Replicas  int
        Image     string
    }
    
    deploy := Deployment{
        Name:      "myapp",
        Namespace: "production",
        Replicas:  3,
        Image:     "myapp:1.0.0",
    }
    
    fmt.Printf("Deploying %s to %s\n", deploy.Name, deploy.Namespace)
}

// ─────────────────────────────────────────────────────────────────
// Functions
// ─────────────────────────────────────────────────────────────────

func deployService(name, version string, replicas int) (bool, error) {
    // Multiple return values
    if replicas < 1 {
        return false, fmt.Errorf("replicas must be at least 1")
    }
    
    log.Printf("Deploying %s:%s with %d replicas", name, version, replicas)
    return true, nil
}

// ─────────────────────────────────────────────────────────────────
// Interfaces
// ─────────────────────────────────────────────────────────────────

type Deployer interface {
    Deploy(name string) error
    Rollback(name string) error
}

type KubernetesDeployer struct {
    Namespace string
}

func (k *KubernetesDeployer) Deploy(name string) error {
    log.Printf("Deploying %s to Kubernetes", name)
    return nil
}

func (k *KubernetesDeployer) Rollback(name string) error {
    log.Printf("Rolling back %s", name)
    return nil
}

// ─────────────────────────────────────────────────────────────────
// Error Handling
// ─────────────────────────────────────────────────────────────────

func readConfig(path string) (map[string]string, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("failed to read config: %w", err)
    }
    
    config := make(map[string]string)
    // Parse config...
    return config, nil
}

// ─────────────────────────────────────────────────────────────────
// Concurrency
// ─────────────────────────────────────────────────────────────────

func checkHealthConcurrently(urls []string) {
    results := make(chan string, len(urls))
    
    for _, url := range urls {
        go func(u string) {
            resp, err := http.Get(u + "/health")
            if err != nil {
                results <- fmt.Sprintf("%s: ERROR - %v", u, err)
                return
            }
            defer resp.Body.Close()
            results <- fmt.Sprintf("%s: %d", u, resp.StatusCode)
        }(url)
    }
    
    for i := 0; i < len(urls); i++ {
        fmt.Println(<-results)
    }
}

// Using sync.WaitGroup
func deployAll(services []string) {
    var wg sync.WaitGroup
    
    for _, svc := range services {
        wg.Add(1)
        go func(s string) {
            defer wg.Done()
            deployService(s)
        }(svc)
    }
    
    wg.Wait()  // Wait for all to complete
}
```

### 3.2 Go Kubernetes Client

```go
package main

import (
    "context"
    "fmt"
    "log"
    "path/filepath"
    
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    "k8s.io/client-go/util/homedir"
)

func main() {
    // Load kubeconfig
    home := homedir.HomeDir()
    kubeconfig := filepath.Join(home, ".kube", "config")
    
    config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
    if err != nil {
        log.Fatalf("Failed to load kubeconfig: %v", err)
    }
    
    // Create clientset
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        log.Fatalf("Failed to create client: %v", err)
    }
    
    // List pods
    ctx := context.Background()
    pods, err := clientset.CoreV1().Pods("default").List(ctx, metav1.ListOptions{})
    if err != nil {
        log.Fatalf("Failed to list pods: %v", err)
    }
    
    for _, pod := range pods.Items {
        fmt.Printf("Pod: %s, Status: %s\n", pod.Name, pod.Status.Phase)
    }
    
    // Scale deployment
    scale, err := clientset.AppsV1().Deployments("default").GetScale(
        ctx, "myapp", metav1.GetOptions{})
    if err != nil {
        log.Fatalf("Failed to get scale: %v", err)
    }
    
    scale.Spec.Replicas = 5
    _, err = clientset.AppsV1().Deployments("default").UpdateScale(
        ctx, "myapp", scale, metav1.UpdateOptions{})
    if err != nil {
        log.Fatalf("Failed to scale: %v", err)
    }
    
    fmt.Println("Scaled myapp to 5 replicas")
}
```

---

## 4. Common DevOps Automation Tasks

### 4.1 Log Parser (Python)

```python
#!/usr/bin/env python3
"""Parse and analyze application logs."""
import re
from collections import Counter
from datetime import datetime

def parse_nginx_logs(log_file: str):
    """Parse NGINX access logs and generate statistics."""
    pattern = r'(\S+) - - \[(.*?)\] "(\S+) (\S+) \S+" (\d+) (\d+)'
    
    stats = {
        "total_requests": 0,
        "status_codes": Counter(),
        "endpoints": Counter(),
        "ips": Counter(),
        "errors": []
    }
    
    with open(log_file, 'r') as f:
        for line in f:
            match = re.match(pattern, line)
            if match:
                ip, timestamp, method, path, status, size = match.groups()
                stats["total_requests"] += 1
                stats["status_codes"][status] += 1
                stats["endpoints"][path] += 1
                stats["ips"][ip] += 1
                
                if int(status) >= 500:
                    stats["errors"].append({
                        "ip": ip, "path": path, 
                        "status": status, "timestamp": timestamp
                    })
    
    # Print report
    print(f"Total Requests: {stats['total_requests']}")
    print(f"\nTop 10 Endpoints:")
    for endpoint, count in stats["endpoints"].most_common(10):
        print(f"  {count:6d} {endpoint}")
    
    print(f"\nStatus Code Distribution:")
    for code, count in sorted(stats["status_codes"].items()):
        print(f"  {code}: {count}")
    
    print(f"\nRecent Errors: {len(stats['errors'])}")
    for error in stats["errors"][-5:]:
        print(f"  {error}")

if __name__ == "__main__":
    parse_nginx_logs("/var/log/nginx/access.log")
```

### 4.2 Health Checker (Go)

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "sync"
    "time"
)

type HealthResult struct {
    URL        string        `json:"url"`
    Status     string        `json:"status"`
    StatusCode int           `json:"status_code,omitempty"`
    Latency    time.Duration `json:"latency_ms"`
    Error      string        `json:"error,omitempty"`
}

func checkHealth(url string, timeout time.Duration) HealthResult {
    start := time.Now()
    result := HealthResult{URL: url}
    
    client := &http.Client{Timeout: timeout}
    resp, err := client.Get(url)
    
    result.Latency = time.Since(start)
    
    if err != nil {
        result.Status = "DOWN"
        result.Error = err.Error()
        return result
    }
    defer resp.Body.Close()
    
    result.StatusCode = resp.StatusCode
    if resp.StatusCode >= 200 && resp.StatusCode < 300 {
        result.Status = "UP"
    } else {
        result.Status = "UNHEALTHY"
    }
    
    return result
}

func checkAllEndpoints(urls []string, timeout time.Duration) []HealthResult {
    results := make([]HealthResult, len(urls))
    var wg sync.WaitGroup
    
    for i, url := range urls {
        wg.Add(1)
        go func(idx int, u string) {
            defer wg.Done()
            results[idx] = checkHealth(u, timeout)
        }(i, url)
    }
    
    wg.Wait()
    return results
}

func main() {
    endpoints := []string{
        "https://api.example.com/health",
        "https://web.example.com/health",
        "https://db.example.com/health",
    }
    
    results := checkAllEndpoints(endpoints, 5*time.Second)
    
    output, _ := json.MarshalIndent(results, "", "  ")
    fmt.Println(string(output))
    
    // Exit with error if any unhealthy
    for _, r := range results {
        if r.Status != "UP" {
            os.Exit(1)
        }
    }
}
```

---

## 5. Building CLI Tools

### 5.1 Python CLI with Click

```python
#!/usr/bin/env python3
"""DevOps CLI tool using Click."""
import click
import yaml
from kubernetes import client, config

@click.group()
@click.option('--kubeconfig', default=None, help='Path to kubeconfig')
@click.pass_context
def cli(ctx, kubeconfig):
    """DevOps CLI for Kubernetes operations."""
    ctx.ensure_object(dict)
    if kubeconfig:
        config.load_kube_config(config_file=kubeconfig)
    else:
        config.load_kube_config()
    ctx.obj['v1'] = client.CoreV1Api()
    ctx.obj['apps'] = client.AppsV1Api()

@cli.command()
@click.option('-n', '--namespace', default='default')
@click.pass_context
def pods(ctx, namespace):
    """List pods in a namespace."""
    pods = ctx.obj['v1'].list_namespaced_pod(namespace)
    for pod in pods.items:
        click.echo(f"{pod.metadata.name}\t{pod.status.phase}")

@cli.command()
@click.argument('deployment')
@click.option('-n', '--namespace', default='default')
@click.option('-r', '--replicas', type=int, required=True)
@click.pass_context
def scale(ctx, deployment, namespace, replicas):
    """Scale a deployment."""
    body = {'spec': {'replicas': replicas}}
    ctx.obj['apps'].patch_namespaced_deployment_scale(
        deployment, namespace, body)
    click.echo(f"Scaled {deployment} to {replicas} replicas")

@cli.command()
@click.argument('deployment')
@click.option('-n', '--namespace', default='default')
@click.pass_context
def restart(ctx, deployment, namespace):
    """Restart a deployment."""
    from datetime import datetime
    body = {
        'spec': {
            'template': {
                'metadata': {
                    'annotations': {
                        'restartedAt': datetime.utcnow().isoformat()
                    }
                }
            }
        }
    }
    ctx.obj['apps'].patch_namespaced_deployment(deployment, namespace, body)
    click.echo(f"Restarted {deployment}")

if __name__ == '__main__':
    cli()
```

### 5.2 Go CLI with Cobra

```go
package main

import (
    "fmt"
    "os"
    
    "github.com/spf13/cobra"
)

var (
    namespace string
    kubeconfig string
)

func main() {
    rootCmd := &cobra.Command{
        Use:   "devops-cli",
        Short: "DevOps CLI for Kubernetes",
    }
    
    rootCmd.PersistentFlags().StringVarP(&namespace, "namespace", "n", "default", "Kubernetes namespace")
    rootCmd.PersistentFlags().StringVar(&kubeconfig, "kubeconfig", "", "Path to kubeconfig")
    
    // pods command
    podsCmd := &cobra.Command{
        Use:   "pods",
        Short: "List pods",
        RunE:  listPods,
    }
    
    // scale command
    scaleCmd := &cobra.Command{
        Use:   "scale [deployment]",
        Short: "Scale a deployment",
        Args:  cobra.ExactArgs(1),
        RunE:  scaleDeployment,
    }
    scaleCmd.Flags().IntP("replicas", "r", 0, "Number of replicas")
    scaleCmd.MarkFlagRequired("replicas")
    
    rootCmd.AddCommand(podsCmd, scaleCmd)
    
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}

func listPods(cmd *cobra.Command, args []string) error {
    // Implementation using client-go
    fmt.Printf("Listing pods in namespace %s\n", namespace)
    return nil
}

func scaleDeployment(cmd *cobra.Command, args []string) error {
    replicas, _ := cmd.Flags().GetInt("replicas")
    deployment := args[0]
    fmt.Printf("Scaling %s to %d replicas\n", deployment, replicas)
    return nil
}
```

---

## 6. Kubernetes Operators

### 6.1 Simple Operator Pattern (Go)

```go
// controller/controller.go
package controller

import (
    "context"
    "time"
    
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    "sigs.k8s.io/controller-runtime/pkg/log"
    
    myappv1 "myapp/api/v1"
)

type MyAppReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

// Reconcile handles the reconciliation loop
func (r *MyAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    logger := log.FromContext(ctx)
    
    // Fetch the MyApp instance
    var myapp myappv1.MyApp
    if err := r.Get(ctx, req.NamespacedName, &myapp); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    logger.Info("Reconciling MyApp", "name", myapp.Name)
    
    // Check if deployment exists
    deployment := &appsv1.Deployment{}
    err := r.Get(ctx, types.NamespacedName{
        Name:      myapp.Name,
        Namespace: myapp.Namespace,
    }, deployment)
    
    if errors.IsNotFound(err) {
        // Create deployment
        deployment = r.buildDeployment(&myapp)
        if err := r.Create(ctx, deployment); err != nil {
            return ctrl.Result{}, err
        }
        logger.Info("Created deployment", "name", deployment.Name)
    }
    
    // Update status
    myapp.Status.Ready = true
    myapp.Status.Replicas = *deployment.Spec.Replicas
    if err := r.Status().Update(ctx, &myapp); err != nil {
        return ctrl.Result{}, err
    }
    
    // Requeue after 30 seconds
    return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
}

func (r *MyAppReconciler) buildDeployment(myapp *myappv1.MyApp) *appsv1.Deployment {
    replicas := int32(myapp.Spec.Replicas)
    
    return &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      myapp.Name,
            Namespace: myapp.Namespace,
            OwnerReferences: []metav1.OwnerReference{
                *metav1.NewControllerRef(myapp, myappv1.GroupVersion.WithKind("MyApp")),
            },
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: &replicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{"app": myapp.Name},
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{"app": myapp.Name},
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  myapp.Name,
                            Image: myapp.Spec.Image,
                            Ports: []corev1.ContainerPort{
                                {ContainerPort: 8080},
                            },
                        },
                    },
                },
            },
        },
    }
}
```

---

## 7. Testing and Best Practices

```
Testing Best Practices:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Python Testing:                                                │
│  ├── pytest for unit tests                                      │
│  ├── pytest-cov for coverage                                    │
│  ├── moto for AWS mocking                                       │
│  └── responses for HTTP mocking                                 │
│                                                                 │
│  Go Testing:                                                    │
│  ├── go test -v ./...                                           │
│  ├── go test -cover                                             │
│  ├── testify for assertions                                     │
│  └── fake client for K8s                                        │
│                                                                 │
│  Code Quality:                                                  │
│  ├── Python: black, isort, flake8, mypy                         │
│  ├── Go: gofmt, golint, go vet                                  │
│  └── Pre-commit hooks                                           │
│                                                                 │
│  Documentation:                                                 │
│  ├── README with examples                                       │
│  ├── Docstrings (Python) / Comments (Go)                        │
│  └── CLI --help text                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# Example test (Python)
import pytest
from unittest.mock import MagicMock, patch

def test_deploy_service():
    with patch('kubernetes.client.AppsV1Api') as mock_api:
        mock_api.return_value.create_namespaced_deployment.return_value = MagicMock()
        
        result = deploy_service("myapp", "v1.0.0", 3)
        
        assert result == True
        mock_api.return_value.create_namespaced_deployment.assert_called_once()
```

---

## Summary

Programming skills are essential for FAANG-level DevOps/SRE:

1. **Python**: Quick scripting, automation, SDK usage
2. **Go**: CLI tools, operators, performance-critical code
3. **Key Libraries**: client-go, kubernetes Python client, cloud SDKs
4. **CLI Tools**: Click (Python), Cobra (Go)
5. **Operators**: Extend Kubernetes with custom controllers
6. **Testing**: Unit tests, mocking, code coverage

Practice coding daily—both LeetCode for interviews and real-world automation scripts.

---

**Next Part**: [Part 16: Multi-Cloud](./DevOps-Complete-Reference-Guide-Part16-MultiCloud.md)
