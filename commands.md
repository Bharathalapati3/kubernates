
kubectl create -f definition.yml

kubectl run redis123 --image=redis

kubectl run nginx --image=nginx --dry-run=client -o yml > definition.yml

kubectl delete pod pod-name

kubctl describe pod pod-name

kubectl apply -f definition.yml

kubectl edit pod pod-name
