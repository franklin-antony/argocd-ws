# ArgoCD Applications

## Active Applications

- `00-namespaces.yaml` - Creates Kong namespaces (kong-cp-stg1-np, kong-dp-stg1-np)
- `01-certificates.yaml` - Manages cert-manager Certificate resources per environment

## Template Files (Not Deployed)

- `02-kong-cp.yaml.template` - Kong Control Plane Helm chart deployment
  - **Status**: Disabled - requires Kong Helm charts and configuration
  - **To enable**: 
    1. Create `my-app/charts/kong-cp/` with Helm chart
    2. Add environment-specific values files (e.g., `stg1.yaml`)
    3. Update repository URL
    4. Rename to `.yaml`

