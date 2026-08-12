### What is namespace

Namespace is the unique location for a given project in the kubernates world. it is used to differentiate the env, or apps and the quotes can be applied at each name space level


kubectl get pods --namespace=dev

kubectl delete pod pod-1 --namespace=dev

kubectl create -f definition.yml --namespace=dev

