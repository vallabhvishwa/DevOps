# DevOps Engineer's Complete Reference Guide
# Part 12: Service Mesh with Istio

---

## Table of Contents

1. [Introduction to Service Mesh](#1-introduction-to-service-mesh)
2. [Istio Architecture](#2-istio-architecture)
3. [Installing Istio](#3-installing-istio)
4. [Traffic Management](#4-traffic-management)
5. [Security (mTLS)](#5-security-mtls)
6. [Observability](#6-observability)
7. [Resilience Patterns](#7-resilience-patterns)
8. [Istio Gateway and Ingress](#8-istio-gateway-and-ingress)
9. [Troubleshooting Istio](#9-troubleshooting-istio)

---

## 1. Introduction to Service Mesh

### 1.1 What is a Service Mesh?

```
Service Mesh Concept:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Without Service Mesh:                                          │
│  ┌─────────┐              ┌─────────┐                           │
│  │ Service │──── HTTP ───►│ Service │                           │
│  │    A    │              │    B    │                           │
│  └─────────┘              └─────────┘                           │
│                                                                 │
│  Problems:                                                      │
│  ├── Each service handles its own: retries, timeouts, TLS       │
│  ├── No uniform observability                                   │
│  ├── Security scattered across services                         │
│  └── Different implementations per language                     │
│                                                                 │
│  With Service Mesh:                                             │
│  ┌─────────────────┐              ┌─────────────────┐           │
│  │   Service A     │              │   Service B     │           │
│  │  ┌───────────┐  │   mTLS      │  ┌───────────┐  │           │
│  │  │    App    │  │              │  │    App    │  │           │
│  │  └───────────┘  │              │  └───────────┘  │           │
│  │  ┌───────────┐  │              │  ┌───────────┐  │           │
│  │  │  Sidecar  │◄─┼──────────────┼─►│  Sidecar  │  │           │
│  │  │  (Envoy)  │  │   Encrypted  │  │  (Envoy)  │  │           │
│  │  └───────────┘  │              │  └───────────┘  │           │
│  └─────────────────┘              └─────────────────┘           │
│                                                                 │
│  Benefits:                                                      │
│  ├── Automatic mTLS between services                            │
│  ├── Unified traffic management (retries, timeouts)             │
│  ├── Built-in observability (traces, metrics)                   │
│  ├── Language-agnostic security                                 │
│  └── No code changes required                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Mesh Features

```
Service Mesh Capabilities:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  TRAFFIC MANAGEMENT:                                            │
│  ├── Load balancing                                             │
│  ├── Traffic splitting (canary, A/B)                            │
│  ├── Traffic mirroring                                          │
│  ├── Circuit breaking                                           │
│  ├── Retries and timeouts                                       │
│  └── Fault injection                                            │
│                                                                 │
│  SECURITY:                                                      │
│  ├── Mutual TLS (mTLS)                                          │
│  ├── Certificate management                                     │
│  ├── Authorization policies                                     │
│  ├── JWT validation                                             │
│  └── Rate limiting                                              │
│                                                                 │
│  OBSERVABILITY:                                                 │
│  ├── Distributed tracing                                        │
│  ├── Metrics collection                                         │
│  ├── Access logging                                             │
│  ├── Service topology                                           │
│  └── Health checking                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Istio Architecture

```
Istio Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CONTROL PLANE (istiod):                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                        istiod                           │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │    │
│  │  │  Pilot   │  │ Citadel  │  │  Galley  │              │    │
│  │  │ (Config) │  │ (Certs)  │  │(Validate)│              │    │
│  │  └──────────┘  └──────────┘  └──────────┘              │    │
│  └─────────────────────────────────────────────────────────┘    │
│         │                │                                      │
│         │ xDS API        │ Certificates                         │
│         ▼                ▼                                      │
│  DATA PLANE (Envoy Sidecars):                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                            │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │   Pod A     │  │   Pod B     │  │   Pod C     │        │  │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │  │
│  │  │ │   App   │ │  │ │   App   │ │  │ │   App   │ │        │  │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │  │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │  │
│  │  │ │ Envoy   │◄┼──┼►│ Envoy   │◄┼──┼►│ Envoy   │ │        │  │
│  │  │ │ Sidecar │ │  │ │ Sidecar │ │  │ │ Sidecar │ │        │  │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  │                                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  INGRESS/EGRESS:                                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  ┌─────────────────┐         ┌─────────────────┐          │  │
│  │  │ Istio Ingress   │         │ Istio Egress    │          │  │
│  │  │    Gateway      │         │    Gateway      │          │  │
│  │  └─────────────────┘         └─────────────────┘          │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Installing Istio

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.0
export PATH=$PWD/bin:$PATH

# Install Istio (demo profile for learning)
istioctl install --set profile=demo -y

# Profiles available:
# - minimal: Minimum components
# - default: Production-ready
# - demo: Full features for demo
# - empty: Nothing, build your own

# Production installation
istioctl install --set profile=default \
  --set meshConfig.accessLogFile=/dev/stdout \
  --set meshConfig.enableTracing=true \
  --set values.pilot.resources.requests.memory=512Mi

# Enable sidecar injection for namespace
kubectl label namespace myapp istio-injection=enabled

# Verify installation
istioctl verify-install
kubectl get pods -n istio-system

# Install addons (Kiali, Prometheus, Grafana, Jaeger)
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system

# Access Kiali dashboard
istioctl dashboard kiali
```

---

## 4. Traffic Management

### 4.1 VirtualService and DestinationRule

```yaml
# VirtualService - Route traffic
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
  namespace: myapp
spec:
  hosts:
  - myapp
  - myapp.example.com
  http:
  - match:
    - headers:
        x-user-type:
          exact: beta
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 3s
      retryOn: 5xx,reset,connect-failure

---
# DestinationRule - Define subsets and policies
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
  namespace: myapp
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    loadBalancer:
      simple: ROUND_ROBIN
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: LEAST_CONN
```

### 4.2 Traffic Mirroring

```yaml
# Mirror traffic for testing
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
        subset: v1
    mirror:
      host: myapp
      subset: v2
    mirrorPercentage:
      value: 100.0
```

### 4.3 Canary Deployment

```yaml
# Gradual rollout with traffic splitting
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
```

---

## 5. Security (mTLS)

### 5.1 Peer Authentication

```yaml
# Strict mTLS for namespace
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: myapp
spec:
  mtls:
    mode: STRICT  # STRICT, PERMISSIVE, DISABLE

---
# Mesh-wide strict mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT

---
# Port-level mTLS (disable for health checks)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: myapp
  namespace: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  mtls:
    mode: STRICT
  portLevelMtls:
    8080:
      mode: PERMISSIVE  # Allow non-mTLS on port 8080
```

### 5.2 Authorization Policy

```yaml
# Allow only specific services to call myapp
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: myapp-policy
  namespace: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/frontend/sa/frontend-sa"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
  - from:
    - source:
        namespaces: ["monitoring"]
    to:
    - operation:
        paths: ["/health", "/metrics"]

---
# Deny all by default, then allow specific
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: myapp
spec:
  {}  # Empty spec = deny all

---
# JWT validation
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
    forwardOriginalToken: true
```

---

## 6. Observability

### 6.1 Metrics with Prometheus

```yaml
# Istio automatically exposes metrics
# Common metrics:
# - istio_requests_total
# - istio_request_duration_milliseconds
# - istio_request_bytes
# - istio_response_bytes

# Prometheus queries
# Request rate per service
sum(rate(istio_requests_total{reporter="destination"}[5m])) by (destination_service_name)

# Error rate
sum(rate(istio_requests_total{reporter="destination",response_code=~"5.."}[5m])) 
/ 
sum(rate(istio_requests_total{reporter="destination"}[5m]))

# P99 latency
histogram_quantile(0.99, 
  sum(rate(istio_request_duration_milliseconds_bucket{reporter="destination"}[5m])) 
  by (destination_service_name, le))
```

### 6.2 Distributed Tracing

```yaml
# Enable tracing in mesh config
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    enableTracing: true
    defaultConfig:
      tracing:
        sampling: 100.0  # 100% sampling
        zipkin:
          address: jaeger-collector.istio-system:9411
```

```bash
# Access Jaeger UI
istioctl dashboard jaeger
```

### 6.3 Access Logs

```yaml
# Enable access logging
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    accessLogFile: /dev/stdout
    accessLogFormat: |
      [%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%"
      %RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT%
      %DURATION% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)%
      "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%"
      "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%"
      "%UPSTREAM_HOST%" %UPSTREAM_CLUSTER%
```

---

## 7. Resilience Patterns

### 7.1 Circuit Breaker

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutive5xxErrors: 5       # Eject after 5 consecutive 5xx
      interval: 10s                 # Check every 10s
      baseEjectionTime: 30s         # Eject for 30s minimum
      maxEjectionPercent: 50        # Max 50% of hosts ejected
      minHealthPercent: 30          # Need 30% healthy hosts
```

### 7.2 Fault Injection

```yaml
# Inject delays and errors for testing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - fault:
      delay:
        percentage:
          value: 10
        fixedDelay: 5s
      abort:
        percentage:
          value: 5
        httpStatus: 503
    route:
    - destination:
        host: myapp
```

---

## 8. Istio Gateway and Ingress

```yaml
# Gateway (entry point)
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: myapp-gateway
  namespace: myapp
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - myapp.example.com
    tls:
      httpsRedirect: true
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - myapp.example.com
    tls:
      mode: SIMPLE
      credentialName: myapp-tls-secret

---
# VirtualService for Gateway
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.example.com
  gateways:
  - myapp-gateway
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: api-service
        port:
          number: 8080
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: frontend
        port:
          number: 80
```

---

## 9. Troubleshooting Istio

```bash
# Check proxy status
istioctl proxy-status

# Describe proxy config
istioctl proxy-config clusters <pod-name>
istioctl proxy-config routes <pod-name>
istioctl proxy-config listeners <pod-name>
istioctl proxy-config endpoints <pod-name>

# Analyze configuration
istioctl analyze
istioctl analyze -n myapp

# Check if sidecar is injected
kubectl get pods -n myapp -o jsonpath='{.items[*].spec.containers[*].name}'

# View Envoy admin
kubectl exec -it <pod> -c istio-proxy -- curl localhost:15000/stats

# Debug logging
istioctl proxy-config log <pod-name> --level debug

# Check mTLS status
istioctl authn tls-check <pod-name>

# Common issues:
# 1. Sidecar not injected → Check namespace label
# 2. 503 errors → Check DestinationRule subsets match deployment labels
# 3. Connection refused → Check service ports match
# 4. mTLS errors → Check PeerAuthentication mode
```

---

## Summary

Service Mesh with Istio provides:

1. **Traffic Management**: VirtualService, DestinationRule for routing
2. **Security**: Automatic mTLS, AuthorizationPolicy, JWT validation
3. **Observability**: Metrics, traces, access logs out of the box
4. **Resilience**: Circuit breaker, retries, timeouts, fault injection
5. **Ingress**: Gateway for external traffic

Key concepts:
- Sidecar proxy (Envoy) handles all network traffic
- Control plane (istiod) configures proxies
- No application code changes required

---

**Next Part**: [Part 13: Advanced Security](./DevOps-Complete-Reference-Guide-Part13-Security.md)
