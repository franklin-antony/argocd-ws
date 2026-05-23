# Dependencies

External dependencies that need to be installed in the cluster before deploying Kong applications.

## Installation Order

1. cert-manager
2. kong-vault-issuer (ClusterIssuer)
3. Then sync ArgoCD applications

---

## cert-manager

**Version**: v1.14.0  
**Purpose**: TLS certificate management for Kong Gateway certificates  
**Source**: https://github.com/cert-manager/cert-manager/releases/tag/v1.14.0

### Installation

```bash
kubectl apply -f cert-manager-v1.14.0.yaml
```

### Verification

```bash
# Wait for cert-manager to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/instance=cert-manager -n cert-manager --timeout=300s

# Verify CRDs are installed
kubectl get crd | grep cert-manager
```

### Required for

- `my-app/argocd/apps/01-certificates.yaml` - Certificate resources for Kong Gateway

---

## kong-vault-issuer

**Type**: ClusterIssuer  
**Purpose**: Certificate issuer for Kong Gateway certificates  
**Status**: ⚠️ Currently using self-signed issuer (dev/testing only)

### Installation

```bash
kubectl apply -f kong-vault-issuer.yaml
```

### Verification

```bash
kubectl get clusterissuer kong-vault-issuer
```

### Production Recommendation

⚠️ **IMPORTANT**: The current `kong-vault-issuer.yaml` uses a self-signed issuer which is suitable for development/testing only.

For **production**, replace with one of:

1. **Vault PKI** (recommended for enterprise):
   ```yaml
   spec:
     vault:
       server: https://vault.example.com
       path: pki/sign/kong-gateway
       auth:
         kubernetes:
           role: cert-manager
           mountPath: /v1/auth/kubernetes
   ```

2. **Let's Encrypt** (for publicly accessible services):
   ```yaml
   spec:
     acme:
       server: https://acme-v02.api.letsencrypt.org/directory
       privateKeySecretRef:
         name: letsencrypt-prod
       solvers:
       - http01:
           ingress:
             class: nginx
   ```

3. **Internal CA** (for internal services):
   ```yaml
   spec:
     ca:
       secretName: kong-ca-key-pair
   ```

---

## Complete Setup Script

```bash
# 1. Install cert-manager
kubectl apply -f cert-manager-v1.14.0.yaml
kubectl wait --for=condition=ready pod -l app.kubernetes.io/instance=cert-manager -n cert-manager --timeout=300s

# 2. Install ClusterIssuer
kubectl apply -f kong-vault-issuer.yaml

# 3. Verify
kubectl get clusterissuer kong-vault-issuer
kubectl get crd | grep cert-manager

# 4. Sync ArgoCD applications
kubectl patch application kong-certificates-stg1 -n argocd --type merge \
  -p '{"operation":{"sync":{"syncStrategy":{"hook":{}},"prune":false}}}'

# 5. Verify certificates
kubectl get certificates -n kong-cp-stg1-np
kubectl get certificates -n kong-dp-stg1-np
```
