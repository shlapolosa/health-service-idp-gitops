# Visual Architecture Tool - GitOps Repository

This repository contains deployment manifests, ArgoCD applications, and OAM applications for the Visual Architecture Maintenance Tool.

## Repository Structure

```
├── applications/           # ArgoCD Applications for GitOps deployment
│   ├── app-of-apps/       # Application of Applications pattern
│   ├── agents/            # AI Agent service applications
│   ├── frontend/          # Frontend application
│   ├── infrastructure/    # Infrastructure components
│   └── orchestration/     # Orchestration service
├── oam/                   # Open Application Model (OAM) applications
│   ├── agents/            # Agent OAM applications
│   ├── frontend/          # Frontend OAM applications
│   ├── infrastructure/    # Infrastructure OAM applications
│   ├── orchestration/     # Orchestration OAM applications
│   ├── components/        # Reusable OAM components
│   ├── policies/          # OAM policies
│   └── workflows/         # OAM workflows
├── charts/                # Helm charts (if needed)
├── environments/          # Environment-specific configurations
│   ├── dev/              # Development environment
│   ├── staging/          # Staging environment
│   └── prod/             # Production environment
└── scripts/              # GitOps utility scripts

## GitOps Workflow

1. **Source Code Changes**: Developers push to `health-service-idp` repository
2. **CI/CD Pipeline**: GitHub Actions builds containers and updates this repository
3. **ArgoCD Sync**: ArgoCD monitors this repository and deploys changes to vcluster
4. **Semantic Versioning**: All deployments use semantic versioning with commit SHA

## Environment Management

- **Development**: Real-time deployments from main branch
- **Staging**: Controlled deployments for testing
- **Production**: Manual approval required for deployments

## Repository Monitoring

ArgoCD watches the following paths:
- `applications/` - ArgoCD Applications
- `oam/` - OAM Applications  
- `environments/*/` - Environment-specific overrides

## Quick Start

```bash
# Apply ArgoCD App-of-Apps
kubectl apply -f applications/app-of-apps/

# Monitor deployments
kubectl get applications -n argocd
```

## Security

This repository contains deployment manifests only. No sensitive data or secrets should be committed here. Use Kubernetes secrets or external secret management systems.

---

**Related Repository**: [health-service-idp](https://github.com/shlapolosa/health-service-idp) - Application source code