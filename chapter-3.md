### What is a ReplicaSet

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. In production you typically run multiple instances of an application so it can handle traffic and remain available if individual pods fail. A ReplicaSet watches the pods that match its selector and will create or remove pods as needed to maintain the desired number of replicas.

Important notes
- The ReplicaSet's selector must match labels on the pod template. If the selector does not match the template labels, the ReplicaSet will not manage its pods.
- Selectors on a ReplicaSet are immutable after creation. If you need to change the selector or update the pod spec in a controlled way, use a Deployment which manages ReplicaSets for you and provides rolling updates and rollbacks.
- In most use cases you should prefer a Deployment over directly creating a ReplicaSet unless you have a specific reason to manage ReplicaSets by hand.

Example ReplicaSet manifest (replicaset.yml)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-set-1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx:1.23
          ports:
            - containerPort: 80
```

Recommended kubectl commands
- Create or update from a file (preferred):
  - kubectl apply -f replicaset.yml
- Create (alternative):
  - kubectl create -f replicaset.yml
- List ReplicaSets:
  - kubectl get replicasets
  - kubectl get rs -o wide
- Inspect a ReplicaSet:
  - kubectl describe rs replica-set-1
  - kubectl get rs replica-set-1 -o yaml
- See pods managed by the ReplicaSet (by label):
  - kubectl get pods -l app=my-app
- Scale a ReplicaSet:
  - kubectl scale rs replica-set-1 --replicas=5
- Edit a ReplicaSet in your editor:
  - kubectl edit rs replica-set-1
- Patch a ReplicaSet (e.g., change labels or fields):
  - kubectl patch rs replica-set-1 --type='merge' -p '{"spec":{"replicas":4}}'
- Delete a ReplicaSet (this also deletes the pods it created):
  - kubectl delete rs replica-set-1

When to use a Deployment instead
A Deployment manages ReplicaSets for you and provides higher-level features such as rolling updates, rollbacks, and declarative updates. If you want to update the pod template or change selectors, create a Deployment; it will create and manage the underlying ReplicaSet(s).

Example: create a Deployment from a manifest
- kubectl apply -f deployment.yml

Examples and quick checks
- After applying the manifest, verify pods are running and the desired replica count is met:
  - kubectl get pods -l app=my-app
  - kubectl get rs
  - kubectl describe rs replica-set-1
