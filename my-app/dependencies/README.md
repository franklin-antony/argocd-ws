# Dependencies

External dependencies that need to be installed in the cluster before deploying Kong applications.

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
