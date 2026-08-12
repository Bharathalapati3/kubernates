# Namespaces in Kubernetes

## What is a Namespace

A Namespace is a virtual cluster inside a Kubernetes cluster. Namespaces provide a scope for names of resources (pods, services, deployments, etc.) and are commonly used to isolate environments (dev, staging, prod), teams, or applications within the same physical cluster. They help with resource organization, access control, and policy application (quotas, limits, RBAC).

Kubernetes ships with a few built-in namespaces:
- default — default for objects without a namespace
- kube-system — system components (kube-dns, kube-proxy, controller-manager, scheduler)
- kube-public — readable by all users (used for cluster info)

Use namespaces when you need logical separation inside one cluster. They do not provide strong multi-tenancy or network isolation by themselves (network policies and RBAC are required for that).

---

## Common kubectl commands for namespaces

Note: the shorthand flag `-n` is equivalent to `--namespace`.

Basic namespace management

- Create a namespace

```bash
kubectl create namespace dev
kubectl create ns dev   # shorthand
```

- Apply a namespace via YAML

```yaml
# namespace-dev.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    env: development
```

Apply it:

```bash
kubectl apply -f namespace-dev.yaml
```

- List namespaces

```bash
kubectl get namespaces
kubectl get ns
```

- Describe a namespace

```bash
kubectl describe namespace dev
```

- Delete a namespace

```bash
kubectl delete namespace dev
kubectl delete ns dev
```

Working with resources in a specific namespace

- List resources in a namespace

```bash
kubectl get pods -n dev
kubectl get all -n dev   # pods, svc, deploy, etc.
```

- Create/apply resources into a namespace (both ways)

```bash
# Using -n flag
kubectl apply -f app-deployment.yaml -n dev

# Or include namespace in the YAML metadata
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev   # resource will be created in `dev`
...
```

- Delete a resource in a namespace

```bash
kubectl delete pod pod-1 -n dev
```

- Describe a resource in a namespace

```bash
kubectl describe pod pod-1 -n dev
```

- Get resources across all namespaces

```bash
kubectl get pods --all-namespaces
kubectl get all -A
```

Set or change the default namespace for your current kubeconfig context

```bash
# Set namespace for the current context (so you can omit -n on future commands)
kubectl config set-context --current --namespace=dev

# Or use a named context
kubectl config set-context my-context --namespace=dev
kubectl config use-context my-context
```

Checking which namespace your context uses

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
# prints the namespace for the current context, blank means `default`
```

Namespacerelated resource controls

- Resource quota (limit how many resources a namespace can use)

```yaml
# quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

Apply it:

```bash
kubectl apply -f quota.yaml
kubectl get resourcequota -n dev
kubectl describe resourcequota compute-resources -n dev
```

- LimitRange (default resource requests/limits within a namespace)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limits
  namespace: dev
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 250m
      memory: 256Mi
    type: Container
```

RBAC and rolebindings within a namespace

- Create Role and RoleBinding scoped to namespace (example)

```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f role.yaml -f rolebinding.yaml
```

Labeling and annotating namespaces

```bash
kubectl label namespace dev team=payments
kubectl annotate namespace dev owner=alice@example.com
kubectl get namespace dev --show-labels
```

Editing or patching a namespace

```bash
# Edit the Namespace resource directly
kubectl edit namespace dev

# Patch (e.g., add a label)
kubectl patch namespace dev -p '{"metadata": {"labels": {"team": "payments"}}}'
```

Other useful commands

- View events in a namespace

```bash
kubectl get events -n dev
```

- Scale a deployment in a namespace

```bash
kubectl scale deployment/my-deployment --replicas=3 -n dev
```

- Exec into a pod in a namespace

```bash
kubectl exec -it pod-1 -n dev -- /bin/sh
```

- Port-forward a pod/service in a namespace

```bash
kubectl port-forward svc/my-service 8080:80 -n dev
```

---

## Example workflow

1. Create a namespace for development:

```bash
kubectl create ns dev
```

2. Set your current context to use the dev namespace so you can omit -n:

```bash
kubectl config set-context --current --namespace=dev
```

3. Deploy an app into the namespace (YAML includes metadata.namespace or use -n):

```bash
kubectl apply -f app-deployment.yaml -n dev
```

4. Inspect resources:

```bash
kubectl get all
kubectl describe pod <pod-name>
```

5. Apply ResourceQuota or LimitRange if you need limits for the namespace.

6. When done, delete the namespace (this deletes all namespace-scoped resources):

```bash
kubectl delete namespace dev
```

Note: Deleting a namespace will remove everything in it — it can take a while if finalizers block deletion.

---

If you want, I can also:
- add examples specific to Deployments/Ingress/Services in a namespace,
- show a full example repository structure with YAML files, or
- add notes about network isolation (NetworkPolicy) and multi-tenancy best practices.
