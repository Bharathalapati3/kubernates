# Secrets

Kubernetes Secrets are objects used to store and manage sensitive information — such as passwords, OAuth tokens, and SSH keys — separately from application code. Secrets let you avoid embedding sensitive data in Pod specs or images.

## Types
- Opaque (default): arbitrary key/value pairs.
- kubernetes.io/tls: certificate and private key (tls.crt, tls.key).
- kubernetes.io/dockerconfigjson: Docker registry credentials (used for imagePullSecrets).
- kubernetes.io/basic-auth, kubernetes.io/ssh-auth, and other well-known types supported by some tooling.

## How data is stored
- Secret data is stored on the API server and persisted in etcd. Values in `data` are base64-encoded, not encrypted by default.
- You can provide `stringData` in manifests or kubectl commands (human-friendly) and Kubernetes will convert them to `data` (base64) on the server.

## Creating Secrets
- From literals:
  kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password='S3cr3t'

- From files:
  kubectl create secret generic tls-secret --from-file=tls.crt=./tls.crt --from-file=tls.key=./tls.key

- TLS secret:
  kubectl create secret tls my-tls --cert=path/to/tls.crt --key=path/to/tls.key

- From Docker config for image pulls:
  kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=USER --docker-password=PASS --docker-email=you@example.com

- Declarative YAML (example using stringData):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
stringData:
  username: admin
  password: S3cr3t
```

## Using Secrets in Pods
- As environment variables (env or envFrom):

```yaml
env:
- name: USERNAME
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: username
```

- Mounted as files in a volume:

```yaml
volumes:
- name: secret-vol
  secret:
    secretName: my-secret
containers:
- name: app
  volumeMounts:
  - name: secret-vol
    mountPath: "/etc/secret"
```

- For pulling images (imagePullSecrets):

```yaml
imagePullSecrets:
- name: regcred
```

## Inspecting and decoding Secrets
- Secrets are base64-encoded in the `data` field. To view a value:

kubectl get secret my-secret -o jsonpath="{.data.password}" | base64 --decode

- To view full YAML (be careful — this exposes secrets):

kubectl get secret my-secret -o yaml

## Updating and rolling pods
- Update a Secret with `kubectl apply -f secret.yaml` or `kubectl create secret --dry-run=client -o yaml ... | kubectl apply -f -`.
- Pods that consume Secrets as files must be restarted to pick up changes (some controllers can trigger rolling updates or you can rollout restart the Deployment).

## Best practices
- Do not store Secrets in source control. Use tools like SealedSecrets, Mozilla SOPS, or external secret managers that integrate with Kubernetes (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) and the Kubernetes External Secrets or Secrets Store CSI driver.
- Enable encryption at rest: configure the API server's EncryptionConfiguration to encrypt Secret data in etcd.
- Limit access with RBAC: restrict which service accounts and users can read Secrets.
- Audit accesses to Secrets and rotate credentials regularly.
- Avoid exposing Secrets in pod logs or environment variables when not necessary; prefer volume mounts with appropriate filesystem permissions.
- Use short-lived credentials and automated rotation where possible.
- For GitOps workflows, keep the encrypted secret artifacts (e.g., sealed secrets or SOPS-encrypted files) in the repo, not raw secrets.

## Integrations and advanced options
- SealedSecrets (bitnami) / kubeseal: encrypt secrets into a sealed secret that can be safely stored in Git and decrypted by the controller in the cluster.
- Secrets Store CSI Driver: mounts external secrets from Vault/KeyVault/SecretsManager directly into the pod as files and can be configured for rotation.
- External controllers/operators (kubernetes-external-secrets) sync secrets from cloud secret managers into Kubernetes when needed.

## Common pitfalls
- Remember that base64 encoding is not encryption — anyone with `get` access to Secrets can decode values.
- By default, etcd may store Secrets unencrypted unless EncryptionConfiguration is configured.
- Some cloud-managed Kubernetes services enable encryption by default; check your cluster settings.
- Large binary secrets may exceed object size limits — keep Secrets small and split if necessary.

## Troubleshooting
- If a Pod can't access a Secret, check:
  - The Secret exists in the same namespace as the Pod.
  - The service account has RBAC permission to read it.
  - The key name referenced matches the key in the Secret.
- Use `kubectl describe secret my-secret` to check metadata and `kubectl logs` for controller errors.

---

References and tools: kubectl, SealedSecrets (kubeseal), SOPS, Secrets Store CSI Driver, HashiCorp Vault, cloud secret managers.
