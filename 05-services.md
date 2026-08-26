# Kubernetes Services

## Overview

A Service in Kubernetes is an abstract way to expose applications running on a set of Pods as a network service. Services provide a stable endpoint for accessing pods, even as pods are created and [...]

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
