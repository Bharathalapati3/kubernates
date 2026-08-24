 Services

 Services in kubertantes is a object to expose the pod to the out side world. It will group the pods by the labels and allocate service to al those pods.

Services are are of Different types, NodePort, ClusterIP and LoadBalancer.

Difference between NodePort, Cluster IP and Load Balancer



Sample Config:

apiVersion: v1
kind: Service
metadata:
  name: sample-service
spec:
  type: ClusterIp
  ports:
    - port: 8080
      targetPort: 8080
      NodePort: 3006
 
