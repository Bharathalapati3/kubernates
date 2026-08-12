### What is Replicaset
  In the real world scenario, you need to have a more than one instance of the app running to serve the seamless traffic to the users. The replicaset feature defines how many instaces
  of the pod required. It auto brings the new pods when there is a pod gets killed and ensure the pods are upto a desired state.


  example definition

  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: replica-set-1
    labels:
      name: replica-set-1
  spec:
    template:
      metadata:
        name: pod-1
        labels:
          name: pod-1
      spec:
        containers:
          - name: container-1
            image: image-name
    replicas: 3
    selector:
      matchLabels:
        name: pod-1

# Example Command

kubectl create -f replicaset.yml

kubectl get replicaset replicaset-name

kubectl delete replicaset replicaset-name
