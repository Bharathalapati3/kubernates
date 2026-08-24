# Certification Tip: Imperative Commands

While you will mostly work the declarative way (using definition files), imperative commands are useful for quick one-off tasks and for generating definition templates that you can save and edit.

Before we begin, familiarize yourself with two options that are handy when running the commands below:

- `--dry-run`:
  - By default, when you run a kubectl command it will create the resource immediately. Use `--dry-run=client` to test the command without creating the resource. This prints the object that would be sent to the API server without actually sending it.

- `-o yaml`:
  - Print the resource definition in YAML format to stdout. Combine this with `--dry-run=client` and shell redirection to generate a manifest file you can edit and apply later.

Example: generate a pod manifest and save to a file

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

---

## Pod

Create an NGINX Pod:

```bash
kubectl run nginx --image=nginx
```

Generate the Pod manifest YAML (don't create it):

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

---

## Deployment

Create a Deployment running NGINX:

```bash
kubectl create deployment nginx --image=nginx
```

Generate the Deployment YAML without creating it:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

Create a Deployment with 4 replicas:

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

Scale an existing Deployment:

```bash
kubectl scale deployment nginx --replicas=4
```

You can also generate a deployment YAML and save it to a file to edit fields (for example to set `replicas` or add labels) before creating it:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
# Edit nginx-deployment.yaml (add/modify replicas, selectors, etc.)
kubectl apply -f nginx-deployment.yaml
```

---

## Service

Create a Service named `redis-service` of type ClusterIP to expose a Pod named `redis` on port 6379 (generate YAML only):

```bash
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml
```

Note: `kubectl expose` will automatically use the pod's labels as the Service selector.

Alternatively, generate a ClusterIP Service manifest using `kubectl create service` (this command assumes a selector of `app=redis` and does not automatically use the pod's labels):

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

Create a NodePort Service named `nginx-service` to expose a Pod's port 80 on node port 30080 (generate YAML only):

```bash
kubectl expose pod nginx --port=80 --name=nginx-service --type=NodePort --dry-run=client -o yaml
```

Note: `kubectl expose` will create a NodePort Service but DOES NOT allow specifying the node port directly in the command. To set a specific node port, generate the YAML, add `nodePort: 30080` under the service's `spec.ports[]`, and then apply the manifest.

Or create a NodePort Service and specify a node port directly:

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

This `kubectl create service nodeport` form does not automatically pick up the pod's labels as selectors.

Both approaches have trade-offs: `kubectl expose` uses the pod labels as selectors (convenient) but won't let you specify `nodePort` on the CLI; `kubectl create service` can set `nodePort` but may not use the pod's labels. My recommendation is to use `kubectl expose` to generate a manifest and then edit the YAML if you need to set `nodePort` or tweak selectors.

---

Reference

- https://kubernetes.io/docs/reference/kubectl/conventions/
