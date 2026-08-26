# Namespaces in Kubernetes

## What is a Namespace

A Namespace is a virtual cluster inside a Kubernetes cluster. Namespaces provide a scope for names of resources (pods, services, deployments, etc.) and are commonly used to isolate environments (de[...]

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

Notes and examples for resource quota, limitrange and RBAC are included in the examples directory.
