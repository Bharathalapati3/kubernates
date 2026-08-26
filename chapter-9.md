# Kubernetes Service Accounts and Tokens with Their Various Versions

## Overview
Service accounts provide an identity for processes running inside a Pod. They are used to authenticate and authorize access to Kubernetes resources. Every Pod runs as a service account, and tokens are used to prove the identity of that service account.

## What Are Service Accounts?

Service accounts are Kubernetes objects that represent an identity within a namespace. They enable Pods and applications to:
- Authenticate with the Kubernetes API server
- Access cluster resources according to configured permissions
- Enable secure inter-pod communication

### Default Service Account
Every namespace has a default service account automatically created:
```bash
kubectl get serviceaccount -n default
kubectl describe sa default
```

## Service Account Tokens - Versions Overview

### Version 1: Legacy Token Format (Pre-1.24)
**Kubernetes Versions:** < 1.24

**Characteristics:**
- Tokens stored as secrets in etcd
- Secret name format: `<service-account-name>-token-xxxxx`
- Long-lived and manually managed
- Token issued at service account creation time

**Token Structure:**
- Format: JWT (JSON Web Token)
- Expiration: None (permanent until manually revoked)
- Mounted at: `/var/run/secrets/kubernetes.io/serviceaccount/token`

**Example:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: default-token-abc123
  namespace: default
type: kubernetes.io/service-account-token
data:
  token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  namespace: ZGVmYXVsdA==
```

**Limitations:**
- No automatic token rotation
- Tokens don't expire
- Difficult to track token usage
- Security risk if token is compromised

---

### Version 2: Bound Service Account Tokens (1.21+)
**Kubernetes Versions:** 1.21 - 1.23 (Beta)

**Key Improvements:**
- Tokens bound to specific workloads (Pods)
- Tokens contain additional metadata
- Foundation for time-bound tokens

**Features:**
- Audience binding: tokens can be bound to specific APIs
- Can be used with workload identity
- Enhanced security model

---

### Version 3: Time-Bound Service Account Tokens (1.22+)
**Kubernetes Versions:** 1.22+ (Beta, GA in 1.24+)

**Major Changes:**
- Short-lived tokens (default: 1 hour)
- Automatic token rotation
- Tokens cannot be mounted as Secrets
- Service account controller generates tokens on-the-fly

**Characteristics:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: production
```

**Token Properties:**
- Lifetime: Configurable (default: 1 hour, min: 10 minutes)
- Auto-rotation: Every 80% of the token's lifetime
- Format: Signed JWT
- Mounting: Via Projected Volume (not as Secret)

**Pod Manifest Example:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  namespace: production
spec:
  serviceAccountName: my-app
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-xxxxx
      readOnly: true
  volumes:
  - name: kube-api-access-xxxxx
    projected:
      sources:
      - serviceAccountToken:
          audience: https://kubernetes.default.svc
          expirationSeconds: 3600
          path: token
      - configMap:
          name: kube-root-ca.crt
          items:
          - key: ca.crt
            path: ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

---

## Comparison Table

| Feature | Version 1 (Legacy) | Version 2 (Bound) | Version 3 (Time-Bound) |
|---------|-------------------|------------------|------------------------|
| **K8s Version** | < 1.24 | 1.21-1.23 | 1.22+ (GA 1.24+) |
| **Lifetime** | Infinite | Long-lived | 1 hour (configurable) |
| **Rotation** | Manual | Manual | Automatic |
| **Storage** | Secret | Secret | Projected Volume |
| **Audience** | None | Optional | Supported |
| **Security** | Lower | Medium | Higher |
| **Expiration** | No | No | Yes |

---

## Token Structure (JWT Breakdown)

A Kubernetes service account token is a JWT with three parts:

```
header.payload.signature

Example:
eyJhbGciOiJSUzI1NiIsImtpZCI6IlhYWFhYWFhYIn0.
eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTY5MzQ3MjAwMCwiYXV0IjoiIiwiaWF0IjoxNjkzNDY4NDAwLCJpc3MiOiJodHRwczovL2t1YmVybmV0ZXMuZGVmYXVsdC5zdmMiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImRlZmF1bHQiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVmYXVsdCIsInVpZCI6ImFiY2QtMTIzNCJ9fSwibmJmIjoxNjkzNDY4NDAwLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZWZhdWx0In0.
signature_here
```

**Header:**
```json
{
  "alg": "RS256",
  "kid": "XXXXXXXX"
}
```

**Payload:**
```json
{
  "aud": ["https://kubernetes.default.svc"],
  "exp": 1693472000,
  "iat": 1693468400,
  "iss": "https://kubernetes.default.svc",
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "default",
      "uid": "abcd-1234"
    }
  },
  "sub": "system:serviceaccount:default:default"
}
```

---

## Accessing Tokens in Pods

### Legacy Approach (Pre-1.24)
```bash
# Read token from mounted secret
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CA_CERT=$(cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt)
NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)

# Use token to access API
curl -H "Authorization: Bearer $TOKEN" \
  --cacert $CA_CERT \
  https://kubernetes.default.svc/api/v1/namespaces/$NAMESPACE/pods
```

### Modern Approach (1.24+)
The token access remains the same from Pod perspective, but:
- Token is automatically rotated
- Mounted via projected volume
- No Secret object created

---

## Migration Path

### Enabling Time-Bound Tokens (1.22+)

The feature gate `ServiceAccountIssuerDiscovery` must be enabled:

```bash
# Check if enabled
kubectl get --raw /api/v1/namespaces/kube-system/services/kube-apiserver-token-issuer

# Enable in API server
--service-account-issuer=https://kubernetes.default.svc
--service-account-key-file=/etc/kubernetes/pki/sa.key
--service-account-signing-key-file=/etc/kubernetes/pki/sa.key
```

### Disabling Legacy Tokens (1.24+)

Set Pod spec to prevent legacy token mounting:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
automountServiceAccountToken: false
---
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: true  # Override at Pod level
  containers:
  - name: app
    image: my-app:latest
```

---

## Best Practices

1. **Use Time-Bound Tokens:** Upgrade to Kubernetes 1.24+ for automatic token rotation
2. **Limit Token Lifetime:** Set `expirationSeconds` to the minimum required (e.g., 3600 for 1 hour)
3. **Disable Unnecessary Tokens:** Set `automountServiceAccountToken: false` when not needed
4. **Rotate Credentials Regularly:** Even with auto-rotation, audit token usage
5. **Use RBAC:** Bind service accounts to minimal required permissions
6. **Monitor Token Usage:** Enable audit logging to track token access
7. **Workload Identity:** Use external identity providers (e.g., IRSA for AWS, Workload Identity for GCP)

---

## Commands Reference

```bash
# List all service accounts
kubectl get serviceaccounts -n <namespace>

# Create service account
kubectl create serviceaccount my-app -n production

# Describe service account
kubectl describe sa my-app -n production

# View token (pre-1.24 only)
kubectl get secret <token-secret-name> -o jsonpath='{.data.token}' | base64 -d

# Check service account mounts
kubectl get pod <pod-name> -o yaml | grep serviceAccountName

# Decode JWT token
echo <token> | cut -d'.' -f2 | base64 -d | jq .
```

---

## Security Considerations

- **Token Leakage:** Short-lived tokens minimize impact of leaked tokens
- **RBAC Integration:** Service accounts must have proper RBAC roles/bindings
- **Audit Logging:** Enable audit logs to track service account usage
- **Network Policies:** Restrict Pod-to-API communication where possible
- **Pod Security Standards:** Use PSS to enforce security policies
