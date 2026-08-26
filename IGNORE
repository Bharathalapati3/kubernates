# Security Context

A security context defines privilege and access control settings for a Pod or Container. It includes settings like:
- User and group IDs
- SELinux options
- Read-only root filesystem
- Linux capabilities
- Privileged mode
- AppArmor and seccomp profiles

## Pod-Level Security Context

Pod-level security contexts apply to all containers within the pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: ['sh', '-c', 'sleep 1h']
```

### Key Pod-Level Fields:

- **runAsUser**: The UID to run the container processes
- **runAsGroup**: The GID to run the container processes
- **fsGroup**: A special supplemental group that applies to all containers in a pod
- **runAsNonRoot**: Enforces that containers run as non-root users
- **seLinuxOptions**: SELinux labels to apply
- **supplementalGroups**: Additional group IDs for processes

## Container-Level Security Context

Container-level security contexts override pod-level settings.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-security-context
spec:
  containers:
  - name: app
    image: nginx:latest
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      runAsUser: 2000
      readOnlyRootFilesystem: true
      capabilities:
        add:
        - NET_BIND_SERVICE
        drop:
        - ALL
```

### Key Container-Level Fields:

- **allowPrivilegeEscalation**: Controls whether a process can gain more privileges than its parent process
- **runAsNonRoot**: Validates that the container runs as a non-root user
- **readOnlyRootFilesystem**: Mounts the root filesystem as read-only
- **privileged**: Runs the container in privileged mode (full access to host resources)
- **capabilities**: Linux capabilities to add or drop

## Linux Capabilities

Linux capabilities provide fine-grained control over privileged operations without giving containers full root access.

### Common Capabilities:

| Capability | Purpose |
|------------|---------|
| CAP_NET_BIND_SERVICE | Bind to ports below 1024 |
| CAP_SYS_ADMIN | System administration operations |
| CAP_CHOWN | Change file owner permissions |
| CAP_DAC_OVERRIDE | Bypass file permission checks |
| CAP_SETUID | Change user ID |
| CAP_SETGID | Change group ID |
| CAP_SYS_PTRACE | Trace processes |
| CAP_NET_RAW | Send raw network packets |

### Adding Capabilities

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: caps-add-demo
spec:
  containers:
  - name: app
    image: ubuntu:20.04
    command: ['sleep', '3600']
    securityContext:
      capabilities:
        add:
        - SYS_TIME       # Allow changing system time
        - NET_ADMIN      # Network administration
```

### Dropping Capabilities

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: caps-drop-demo
spec:
  containers:
  - name: app
    image: nginx:latest
    securityContext:
      capabilities:
        drop:
        - ALL            # Drop all capabilities
        add:
        - NET_BIND_SERVICE  # Add back only what's needed
```

## SELinux Context

SELinux (Security Enhanced Linux) provides an additional layer of security control.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selinux-demo
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
      role: "sysadm_r"
      type: "svirt_apache_t"
      user: "sysadm_u"
  containers:
  - name: app
    image: nginx:latest
```

## Privileged vs Non-Privileged Containers

### Non-Privileged (Recommended)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-privileged
spec:
  containers:
  - name: app
    image: nginx:latest
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      readOnlyRootFilesystem: true
```

### Privileged (Use with Caution)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-demo
spec:
  containers:
  - name: app
    image: ubuntu:20.04
    securityContext:
      privileged: true
```

## Read-Only Root Filesystem

Prevents containers from modifying the root filesystem, improving security.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-fs-demo
spec:
  containers:
  - name: app
    image: nginx:latest
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
```

## Complete Security Context Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-best-practices
spec:
  serviceAccountName: app-account
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    imagePullPolicy: Always
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    resources:
      limits:
        cpu: "100m"
        memory: "128Mi"
      requests:
        cpu: "50m"
        memory: "64Mi"
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /var/cache
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

## Security Context Best Practices

1. **Always run as non-root**: Set `runAsNonRoot: true`
2. **Drop all capabilities**: Use `drop: [ALL]` and add back only what's needed
3. **Use read-only root filesystem**: Set `readOnlyRootFilesystem: true`
4. **Prevent privilege escalation**: Set `allowPrivilegeEscalation: false`
5. **Use seccomp profiles**: Apply runtime default or custom seccomp profiles
6. **Run with resource limits**: Define CPU and memory limits
7. **Use image pull policies**: Set `imagePullPolicy: Always` for security
8. **Implement network policies**: Restrict network traffic between pods
9. **Use pod security policies**: Enforce security standards at the cluster level
10. **Regular auditing**: Monitor and audit security context usage

## Troubleshooting Security Context Issues

### Permission Denied Errors

```bash
# Check pod security context
kubectl describe pod <pod-name>

# View logs for permission issues
kubectl logs <pod-name>

# Verify running user
kubectl exec <pod-name> -- id
```

### Capability Issues

```bash
# View effective capabilities
kubectl exec <pod-name> -- capsh --print
```

### File System Access Issues

```yaml
# Ensure volume permissions match container user
volumeMounts:
- name: data
  mountPath: /data
  subPathExpr: $(POD_NAME)
```

## References

- [Kubernetes Security Context Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Linux Capabilities Man Pages](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
