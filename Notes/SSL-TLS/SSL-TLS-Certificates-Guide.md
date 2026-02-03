# SSL/TLS & Certificates - Complete Guide
## Encryption, Certificates, PKI, Troubleshooting

---

# 1. UNDERSTANDING SSL/TLS

## What Is TLS?

```
TLS = Transport Layer Security (successor to SSL)

PURPOSE:
┌─────────────────────────────────────────────────────────────────┐
│ 1. ENCRYPTION:    Data cannot be read by attackers            │
│ 2. AUTHENTICATION: Verify server (and optionally client)       │
│ 3. INTEGRITY:     Data cannot be modified in transit          │
└─────────────────────────────────────────────────────────────────┘

VERSION HISTORY:
┌─────────────────────────────────────────────────────────────────┐
│ SSL 2.0, 3.0:    Deprecated, insecure                          │
│ TLS 1.0, 1.1:    Deprecated (2020)                             │
│ TLS 1.2:         Current standard                              │
│ TLS 1.3:         Latest, faster, more secure                   │
└─────────────────────────────────────────────────────────────────┘
```

## TLS Handshake

```
TLS 1.2 HANDSHAKE:
┌─────────┐                              ┌─────────┐
│ Client  │                              │ Server  │
└────┬────┘                              └────┬────┘
     │                                        │
     │  1. ClientHello                        │
     │  (supported ciphers, TLS version)      │
     │───────────────────────────────────────►│
     │                                        │
     │  2. ServerHello                        │
     │  (chosen cipher, certificate)          │
     │◄───────────────────────────────────────│
     │                                        │
     │  3. Certificate Validation             │
     │  (verify chain, check expiry)          │
     │                                        │
     │  4. Key Exchange                       │
     │  (establish shared secret)             │
     │───────────────────────────────────────►│
     │                                        │
     │  5. Finished                           │
     │◄──────────────────────────────────────►│
     │                                        │
     │  === Encrypted Communication ===       │
     │◄──────────────────────────────────────►│

TLS 1.3 HANDSHAKE (Faster - 1 RTT):
┌─────────┐                              ┌─────────┐
│ Client  │                              │ Server  │
└────┬────┘                              └────┬────┘
     │  ClientHello + Key Share               │
     │───────────────────────────────────────►│
     │  ServerHello + Key Share + Cert        │
     │◄───────────────────────────────────────│
     │  === Encrypted ===                     │
```

---

# 2. CERTIFICATES

## What Is a Certificate?

```
CERTIFICATE CONTAINS:
┌─────────────────────────────────────────────────────────────────┐
│ Subject:          CN=www.example.com                           │
│ Issuer:           CN=DigiCert CA                               │
│ Validity:         Not Before: Jan 1, 2024                      │
│                   Not After:  Jan 1, 2025                      │
│ Public Key:       RSA 2048-bit                                 │
│ Signature:        sha256WithRSAEncryption                      │
│ Extensions:                                                     │
│   - Subject Alt Names (SANs): www.example.com, example.com     │
│   - Key Usage: Digital Signature, Key Encipherment             │
│   - Extended Key Usage: TLS Web Server Authentication          │
└─────────────────────────────────────────────────────────────────┘
```

## Certificate Types

```
BY VALIDATION LEVEL:
┌─────────────────────────────────────────────────────────────────┐
│ DV (Domain Validation):                                        │
│ - Proves domain ownership                                      │
│ - Cheapest, fastest                                            │
│ - Good for: Most websites                                      │
│                                                                 │
│ OV (Organization Validation):                                  │
│ - Verifies organization exists                                 │
│ - Shows company name in cert                                   │
│ - Good for: Business sites                                     │
│                                                                 │
│ EV (Extended Validation):                                      │
│ - Rigorous verification                                        │
│ - Green bar (historically)                                     │
│ - Good for: Financial institutions                             │
└─────────────────────────────────────────────────────────────────┘

BY COVERAGE:
┌─────────────────────────────────────────────────────────────────┐
│ Single Domain:    www.example.com                              │
│ Wildcard:         *.example.com                                │
│ Multi-Domain (SAN): www.example.com, api.example.com           │
└─────────────────────────────────────────────────────────────────┘
```

## Certificate Chain

```
CERTIFICATE CHAIN (TRUST HIERARCHY):
┌─────────────────────────────────────────────────────────────────┐
│                    Root CA Certificate                         │
│                 (Self-signed, in trust store)                  │
│                           │                                     │
│                           ▼                                     │
│                 Intermediate CA Certificate                    │
│               (Signed by Root, issues certs)                   │
│                           │                                     │
│                           ▼                                     │
│                  End-Entity Certificate                        │
│              (Your server certificate)                         │
└─────────────────────────────────────────────────────────────────┘

CHAIN VALIDATION:
1. Receive server certificate
2. Find issuer (intermediate CA)
3. Verify intermediate signature
4. Find issuer of intermediate (root CA)
5. Verify root is in trust store
6. Check all certs are valid (not expired, not revoked)
```

---

# 3. OPENSSL COMMANDS

## Certificate Operations

```bash
# Generate private key
openssl genrsa -out private.key 2048
openssl genrsa -aes256 -out private.key 4096  # With password

# Generate CSR (Certificate Signing Request)
openssl req -new -key private.key -out request.csr \
  -subj "/C=US/ST=CA/L=SF/O=Company/CN=www.example.com"

# Generate self-signed certificate
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout private.key \
  -out certificate.crt \
  -subj "/CN=www.example.com"

# View certificate details
openssl x509 -in certificate.crt -text -noout

# View CSR details
openssl req -in request.csr -text -noout

# Check private key
openssl rsa -in private.key -check

# Verify certificate matches private key
openssl x509 -noout -modulus -in certificate.crt | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5
# Must match!

# Verify certificate chain
openssl verify -CAfile ca-bundle.crt certificate.crt
```

## Format Conversion

```bash
# PEM to DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER to PEM
openssl x509 -in cert.der -inform DER -out cert.pem

# PEM to PKCS12 (PFX)
openssl pkcs12 -export \
  -out certificate.pfx \
  -inkey private.key \
  -in certificate.crt \
  -certfile ca-bundle.crt

# PKCS12 to PEM
openssl pkcs12 -in certificate.pfx -out certificate.pem -nodes

# Extract cert from PKCS12
openssl pkcs12 -in certificate.pfx -clcerts -nokeys -out cert.pem

# Extract key from PKCS12
openssl pkcs12 -in certificate.pfx -nocerts -nodes -out key.pem
```

## Testing Connections

```bash
# Test SSL connection
openssl s_client -connect www.example.com:443

# Show full certificate chain
openssl s_client -connect www.example.com:443 -showcerts

# Test specific TLS version
openssl s_client -connect www.example.com:443 -tls1_2
openssl s_client -connect www.example.com:443 -tls1_3

# Check certificate expiry
openssl s_client -connect www.example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# Check SNI (Server Name Indication)
openssl s_client -connect www.example.com:443 -servername www.example.com
```

---

# 4. CERTIFICATES IN KUBERNETES

## TLS Secrets

```yaml
# Create TLS secret
apiVersion: v1
kind: Secret
metadata:
  name: my-tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

```bash
# Create TLS secret from files
kubectl create secret tls my-tls-secret \
  --cert=certificate.crt \
  --key=private.key

# View secret
kubectl get secret my-tls-secret -o yaml
```

## Ingress TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  tls:
  - hosts:
    - www.example.com
    secretName: my-tls-secret
  rules:
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

## cert-manager (Automatic Certificates)

```yaml
# ClusterIssuer for Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

---
# Certificate request
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-cert
spec:
  secretName: example-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: www.example.com
  dnsNames:
  - www.example.com
  - example.com
```

---

# 5. AZURE KEY VAULT CERTIFICATES

```bash
# Create certificate in Key Vault
az keyvault certificate create \
  --vault-name myKeyVault \
  --name myCert \
  --policy "$(az keyvault certificate get-default-policy)"

# Import certificate
az keyvault certificate import \
  --vault-name myKeyVault \
  --name myCert \
  --file certificate.pfx \
  --password "pfxpassword"

# Download certificate
az keyvault certificate download \
  --vault-name myKeyVault \
  --name myCert \
  --file downloaded.pem

# Get secret (includes private key)
az keyvault secret show \
  --vault-name myKeyVault \
  --name myCert \
  --query value -o tsv | base64 -d > cert-with-key.pem

# Auto-renew with Key Vault
# Configure certificate policy with auto-renew
```

---

# 6. TROUBLESHOOTING

## Common Issues

### Certificate Expired
```bash
# Check expiry
openssl x509 -in cert.crt -noout -dates
openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -enddate

# Solution: Renew certificate
```

### Certificate Chain Incomplete
```
Error: unable to get local issuer certificate
```

```bash
# Check chain
openssl s_client -connect example.com:443 -showcerts

# Solution: Include intermediate certificates
cat server.crt intermediate.crt > fullchain.crt
```

### Certificate Name Mismatch
```
Error: hostname does not match certificate
```

```bash
# Check certificate names
openssl x509 -in cert.crt -noout -text | grep -A1 "Subject Alternative Name"

# Solution: Use correct domain or get new cert with correct SANs
```

### Private Key Doesn't Match Certificate
```bash
# Verify match
openssl x509 -noout -modulus -in cert.crt | openssl md5
openssl rsa -noout -modulus -in key.key | openssl md5

# Must be identical - if not, key and cert don't match
```

### SSL Handshake Failure
```bash
# Debug handshake
openssl s_client -connect example.com:443 -debug

# Check supported protocols
nmap --script ssl-enum-ciphers -p 443 example.com

# Common causes:
# - TLS version mismatch
# - Cipher suite mismatch
# - SNI required but not sent
```

### Certificate Revoked
```bash
# Check OCSP
openssl s_client -connect example.com:443 -status 2>/dev/null | \
  grep -A 10 "OCSP Response"

# Check CRL
openssl verify -crl_check -CAfile ca.crt cert.crt
```

---

# 7. BEST PRACTICES

```
CERTIFICATE MANAGEMENT:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use 2048-bit RSA or 256-bit ECDSA keys                      │
│ ✓ Use SHA-256 or stronger signatures                          │
│ ✓ Include all required SANs                                   │
│ ✓ Automate certificate renewal                                │
│ ✓ Monitor certificate expiry                                  │
│ ✓ Store private keys securely                                 │
│ ✓ Use secrets management (Key Vault, Secrets Manager)         │
│ ✓ Rotate certificates before expiry                           │
└─────────────────────────────────────────────────────────────────┘

TLS CONFIGURATION:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use TLS 1.2 minimum, prefer TLS 1.3                         │
│ ✓ Disable weak ciphers                                        │
│ ✓ Enable HSTS (HTTP Strict Transport Security)                │
│ ✓ Use forward secrecy (ECDHE)                                 │
│ ✓ Enable OCSP stapling                                        │
│ ✓ Use certificate pinning for mobile apps                     │
└─────────────────────────────────────────────────────────────────┘
```
