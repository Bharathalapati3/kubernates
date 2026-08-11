# What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. A Pod represents one or more containers that are tightly coupled and run on the same node. Containers in a Pod share the same network namespace (including IP address and localhost), can share storage volumes, and have a shared lifecycle.

Key points

- Single or multiple containers: Most Pods contain a single primary container, but a Pod can hold multiple containers that must run together (for example, a main application container plus helper containers).
- Shared networking: All containers in a Pod share the Pod IP and network ports. Containers communicate with each other over localhost.
- Shared storage: Pods can mount Volumes that are visible to all containers in the Pod.
- Ephemeral lifecycle: Pods are meant to be ephemeral. Use higher-level controllers (Deployments, ReplicaSets, StatefulSets, Jobs) to manage Pod replicas, updates, and restarts.

Why run multiple containers in a Pod?

- Sidecar pattern: Run helper containers (logging agents, proxies, or sync agents) alongside the main app.
- Adapter pattern: Transform or adapt data for the main container.
- Ambassador pattern: Handle communication between the Pod and external services.

Common kubectl commands for Pods

- Run a quick Pod (for experimentation):
  kubectl run nginx --image=nginx --restart=Never

  Note: `kubectl run` is useful for short demos. For production, use manifests and controllers.

- Create or apply from a YAML manifest:
  kubectl apply -f pod.yaml

- List Pods:
  kubectl get pods
  kubectl get pods -o wide   # shows node and pod IP

- Describe a Pod (details and events):
  kubectl describe pod <pod-name>

- View logs of a container:
  kubectl logs <pod-name>                 # single-container Pod
  kubectl logs <pod-name> -c <container>  # specific container in a multi-container Pod

- Open an interactive shell inside a container:
  kubectl exec -it <pod-name> -- /bin/sh
  kubectl exec -it <pod-name> -c <container> -- /bin/bash

- Delete a Pod:
  kubectl delete pod <pod-name>

- Get recent events (useful for troubleshooting):
  kubectl get events --sort-by=.metadata.creationTimestamp

Example Pod manifest (pod.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
```

When to use controllers instead of bare Pods

You should usually create Pods through controllers (for example, Deployment) in production. Controllers maintain the desired state, provide automatic restarts, scaling, and rolling updates. Creating a bare Pod is appropriate for simple tests or one-off tasks.

Further reading

- Kubernetes Pods concept: https://kubernetes.io/docs/concepts/workloads/pods/
