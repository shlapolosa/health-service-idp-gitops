# health-service-idp GitOps Repository

This repository contains the GitOps deployment manifests for the health-service-idp application container.

## Structure



## Deployment Flow

1. Source repository CI/CD builds and pushes images
2. Source CI/CD sends repository_dispatch to this repo
3. GitOps workflow updates manifest files with new image tags
4. ArgoCD detects changes and deploys to cluster

## ArgoCD Applications

- **App of Apps**:  - Main application managing all microservices
- **ApplicationSet**: Automatically creates ArgoCD apps for each service in 

## Adding New Services

When ApplicationClaim creates a new microservice, it automatically:
1. Creates a directory in 
2. Generates Knative service and OAM component manifests
3. Updates the ApplicationSet to include the new service
