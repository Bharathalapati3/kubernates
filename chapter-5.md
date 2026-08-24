# Kubernetes Services

## Overview

A Service in Kubernetes is an abstract way to expose applications running on a set of Pods as a network service. Services provide a stable endpoint for accessing pods, even as pods are created and destroyed. They act as a load balancer and network abstraction layer between the external world and your pods.

**Key Purpose:**
- Expose pods to the outside world or other pods within the cluster
- Group pods using label selectors
- Provide a stable IP address and DNS name for accessing pods
- Enable load balancing across multiple pod replicas
- Decouple the client from the specific pod instances

---

## How Services Work

1. **Label Selection**: Services use label selectors to identify which pods to route traffic to
2. **Endpoint Discovery**: Kubernetes automatically discovers all pods matching the selector and creates endpoints
3. **Traffic Routing**: When a request comes to the service, it's distributed to one of the selected pods
4. **Load Balancing**: Services automatically load balance traffic across all matching pods

**Example Flow:**
```
Client/User
    ↓
Service (stable endpoint)
    ↓
Load Balancing
    ↓
Pod 1 | Pod 2 | Pod 3 (selected by labels)
```

---

## Types of Services

### 1. **ClusterIP** (Default)
- **What it is**: Exposes the service only within the cluster
- **Use Case**: Internal communication between pods/services within the same cluster
- **IP Accessibility**: Only accessible from within the cluster using the cluster DNS name
- **DNS Format**: `<service-name>.<namespace>.svc.cluster.local`
- **Typical Use**: Backend services, databases, internal APIs
- **Port Exposure**: No external port needed

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
```

---

### 2. **NodePort**
- **What it is**: Exposes the service on a static port on each node
- **Use Case**: Accessing the service from outside the cluster using node IP and a specific port
- **Port Range**: Node ports are allocated in the range 30000-32767 (customizable)
- **Accessibility**: `<NodeIP>:<NodePort>` or `<ServiceName>:<NodePort>` from outside
- **Typical Use**: Development, testing, external access without a load balancer
- **Advantage**: Simple external access without cloud provider integration
- **Disadvantage**: Exposes cluster nodes and uses high port numbers

**How it Works:**
```
External User
    ↓
NodeIP:30006 (on any cluster node)
    ↓
NodePort Service
    ↓
Routes to backend pod on port 8080
```

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 8080          # Service port (internal)
      targetPort: 8080    # Pod container port
      nodePort: 30006     # External port on nodes (optional, auto-assigned if omitted)
```

**Accessing the Service:**
```bash
# From outside the cluster
curl http://<any-node-ip>:30006

# Example
curl http://192.168.1.10:30006
```

---

### 3. **LoadBalancer**
- **What it is**: Exposes the service using a cloud provider's load balancer
- **Use Case**: Production environments with cloud providers (AWS, GCP, Azure)
- **External IP**: Assigns an external IP address from the cloud provider
- **Accessibility**: `<ExternalIP>:<Port>` from anywhere on the internet
- **Typical Use**: Production services, public-facing APIs, web applications
- **Advantages**: Industry-standard load balancing, automatic failover
- **Disadvantage**: Requires cloud provider integration, may incur costs

**How it Works:**
```
Internet User
    ↓
Cloud LoadBalancer (external IP)
    ↓
LoadBalancer Service
    ↓
Routes to backend pods
```

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: public-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80          # Service port
      targetPort: 8080  # Pod container port
      protocol: TCP
```

**After Creation:**
```bash
kubectl get svc public-service
# NAME              TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
# public-service    LoadBalancer   10.0.0.50       203.0.113.100    80:30500/TCP
```

---

## Comparison: NodePort vs ClusterIP vs LoadBalancer

| Feature | ClusterIP | NodePort | LoadBalancer |
|---------|-----------|----------|--------------|
| **Internal Access** | ✅ Yes | ✅ Yes | ✅ Yes |
| **External Access** | ❌ No | ✅ Yes | ✅ Yes |
| **Access Method** | Cluster DNS | Node IP + Port | External IP |
| **Port Range** | Any | 30000-32767 | Any |
| **Use Case** | Internal services | Dev/Test | Production |
| **Cloud Integration** | Not needed | Not needed | Required |
| **Cost** | No | No | May apply |
| **Complexity** | Low | Medium | Medium-High |

---

## Service Configuration Details

### Key Concepts

#### **Selector**
- Specifies which pods to route traffic to using labels
- Service dynamically discovers all matching pods

#### **Port**
- The port number on which the service listens
- Can be any valid port number

#### **TargetPort**
- The port number on the pod container that receives the traffic
- Must match the port your application listens on

#### **Protocol**
- Defines the protocol: TCP (default) or UDP
- SCTP also supported on some Kubernetes versions

#### **NodePort** (only for NodePort type)
- The static port on each cluster node
- Auto-assigned from 30000-32767 if not specified
- Required for external access in NodePort services

---

## Complete Service Examples

### Example 1: ClusterIP Service (Internal)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database-service
  namespace: default
  labels:
    app: database
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
```

**Usage:**
```bash
# From another pod in the cluster
psql -h database-service:5432 -U postgres
# or with full DNS name
psql -h database-service.default.svc.cluster.local:5432 -U postgres
```

---

### Example 2: NodePort Service (External Testing)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

**Usage:**
```bash
# Get node IP
kubectl get nodes -o wide

# Access from external machine
curl http://<node-ip>:30080
```

---

### Example 3: LoadBalancer Service (Production)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: production
  annotations:
    cloud.google.com/load-balancer-type: "Internal"  # Cloud-specific
spec:
  type: LoadBalancer
  selector:
    app: webapp
    tier: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  sessionAffinity: ClientIP  # Optional: maintain session
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

**Usage:**
```bash
# Get external IP (wait for cloud provider to assign)
kubectl get svc app-service -w

# Access the service
curl http://<external-ip>
```

---

## Common Service Operations

### Create a Service
```bash
kubectl apply -f service.yaml
```

### List Services
```bash
kubectl get svc
# or
kubectl get services
```

### Describe Service Details
```bash
kubectl describe svc service-name
```

### View Service Endpoints
```bash
kubectl get endpoints service-name
```

### Delete a Service
```bash
kubectl delete svc service-name
```

### Edit a Service
```bash
kubectl edit svc service-name
```

---

## Best Practices

1. **Use Label Selectors Consistently**: Ensure pod labels match service selectors
2. **Name Services Descriptively**: Use clear, descriptive service names
3. **Choose Appropriate Service Type**:
   - Internal services → ClusterIP
   - External testing → NodePort
   - Production external → LoadBalancer
4. **Use Namespaces**: Organize services in appropriate namespaces
5. **Define Resource Policies**: Set request/limit policies for pods
6. **Monitor Service Endpoints**: Ensure pods are properly selected
7. **Use DNS Names**: Prefer `service-name:port` over IP addresses

---

## Troubleshooting Services

### Service has no endpoints
```bash
# Check if selector matches pod labels
kubectl get pods --show-labels
kubectl describe svc service-name
```

### Cannot access service
```bash
# Verify service exists and has endpoints
kubectl get svc
kubectl get endpoints service-name

# Check pod connectivity
kubectl exec -it <pod-name> -- curl http://service-name:port
```

### External IP shows <pending>
```bash
# For LoadBalancer type, wait for cloud provider to assign
kubectl get svc -w
# May take several minutes on cloud providers
```

---

## Summary

Kubernetes Services provide a stable, scalable way to expose applications. Choose the right service type based on your needs:
- **ClusterIP**: Internal cluster communication
- **NodePort**: External access in development/testing environments
- **LoadBalancer**: Production-grade external access with cloud integration

Proper service configuration ensures reliable communication between components of your Kubernetes cluster and with the outside world.
