# OAM Workflows

This directory contains KubeVela workflow definitions for managing application lifecycle.

## Available Workflows

-  - Handles OAM → Crossplane ApplicationClaim creation
-  - Manages component dependencies and ordering
-  - Handles failure scenarios and cleanup

## Usage

Workflows are automatically triggered when OAM Applications are created or updated.
The system handles:

1. **Infrastructure Provisioning**: Databases, caches, messaging systems
2. **Application Deployment**: Microservices, frontends, static sites
3. **Pipeline Orchestration**: Data processing workflows
4. **Dependency Management**: Proper ordering and readiness checks
5. **Failure Recovery**: Cleanup and retry mechanisms
