# Install external secrets operator in your k8s cluster using helm
```
helm repo add external-secrets https://charts.external-secrets.io

helm install external-secrets \
   external-secrets/external-secrets \
    -n external-secrets \
    --create-namespace \
  # --set installCRDs=false
```

# vault setup in minikube
```
https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-minikube-raft
```
# Example connect vault using token
```
apiVersion: external-secrets.io/v1beta1
kind: clusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "http://my.vault.server:8200"
      path: "secret"
      # Version is the Vault KV secret engine version.
      # This can be either "v1" or "v2", defaults to "v2"
      version: "v2"
      auth:
        # points to a secret that contains a vault token
        # https://www.vaultproject.io/docs/auth/token
        tokenSecretRef:
          name: "vault-token"
          key: "token"
---
apiVersion: v1
kind: Secret
metadata:
  name: vault-token
data:
  token: cm9vdA== # "root"
```

# Example connect vault using approle
```
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: example
spec:
  provider:
    vault:
      server: "https://vault.acme.org"
      path: "secret"
      version: "v2"
      auth:
        # VaultAppRole authenticates with Vault using the
        # App Role auth mechanism
        # https://www.vaultproject.io/docs/auth/approle
        appRole:
          # Path where the App Role authentication backend is mounted
          path: "approle"
          # RoleID configured in the App Role authentication backend
          roleId: "db02de05-fa39-4855-059b-67221c5c2f63"
          # Reference to a key in a K8 Secret that contains the App Role SecretId
          secretRef:
            name: "my-secret"
            key: "secret-id"
```

# Example secretStore crd to connect AWS secret manager(secret provider)
```
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: secretstore-sample
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: awssm-secret
            key: access-key
          secretAccessKeySecretRef:
            name: awssm-secret
            key: secret-access-key
```
