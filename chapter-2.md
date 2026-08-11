# What is POD ? 
 POD is the smallest deployment unit in the kubernates. Each pod can container more than 1 container. Additional container include helper container to live along side the applicaton container.
 Containers with in the pod can communicate each other easily with the local host, share thesame network.

Deploying the pods

kubectl run nginx --image nginx

kubectl get pods



 
