# Kubernetes ConfigMap

ConfigMap is a Kubernetes resource used to store non-confidential configuration data in key-value pairs. It allows you to decouple configuration from application code and manage environment variabl[...]

## What is a ConfigMap?

A ConfigMap is an API object used to store small amounts of non-sensitive, structured data in the form of key-value pairs. ConfigMaps are useful for:
- Storing environment variables
- Storing configuration files
- Storing command-line arguments
- Managing application settings across different environments

**Note:** ConfigMaps are NOT suitable for sensitive data like passwords or API keys. Use Kubernetes Secrets for that purpose.

## Ways to Create ConfigMap

### 1. Imperative Method

The imperative method involves using `kubectl` commands directly to create ConfigMaps without defining YAML files.

#### From literal values:
```bash
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```

#### From a file:
```bash
kubectl create configmap my-config --from-file=config.properties
```

#### From a directory:
```bash
kubectl create configmap my-config --from-file=/path/to/directory
```

#### Example:
```bash
kubectl create configmap app-config --from-literal=DATABASE_HOST=localhost --from-literal=DATABASE_PORT=5432 --from-literal=LOG_LEVEL=INFO
```

### 2. Declarative Method

The declarative method involves creating a YAML manifest file and applying it using `kubectl apply`.

#### Basic ConfigMap YAML:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  DATABASE_HOST: localhost
  DATABASE_PORT: "5432"
  LOG_LEVEL: INFO
  APP_NAME: MyApplication
```

#### ConfigMap with file content:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-files
  namespace: default
data:
  config.properties: |
    database.url=jdbc:mysql://localhost:3306/mydb
    database.user=admin
    database.password=secret123
  application.yml: |
    server:
      port: 8080
    logging:
      level: INFO
```

Apply the ConfigMap:
```bash
kubectl apply -f configmap.yaml
```

## Using ConfigMap in Pods

### Method 1: Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST
    - name: DATABASE_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_PORT
```

### Method 2: All ConfigMap Keys as Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
```

### Method 3: Volume Mount
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config-files
```

## Viewing and Managing ConfigMaps

### List all ConfigMaps:
```bash
kubectl get configmaps
```

### View ConfigMap details:
```bash
kubectl describe configmap app-config
```

### View ConfigMap in YAML format:
```bash
kubectl get configmap app-config -o yaml
```

### Delete a ConfigMap:
```bash
kubectl delete configmap app-config
```

## Best Practices

1. **Size Limit:** ConfigMaps are limited to 1MB in size
2. **Security:** Don't store sensitive data in ConfigMaps; use Secrets instead
3. **Namespacing:** ConfigMaps are namespace-specific
4. **Immutability:** Consider using `immutable: true` for ConfigMaps that shouldn't change
5. **Version Control:** Always store ConfigMap YAML files in version control
6. **Naming Convention:** Use clear, descriptive names for ConfigMaps

## ConfigMap with Immutability

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-immutable
data:
  LOG_LEVEL: INFO
immutable: true
```

## Summary

| Aspect | Imperative | Declarative |
|--------|-----------|-------------|
| Command | `kubectl create configmap` | `kubectl apply -f configmap.yaml` |
| YAML Required | No | Yes |
| Version Control | Harder to manage | Easy to track |
| Repeatability | Less reliable | More reliable |
| Use Case | Quick testing | Production deployments |
