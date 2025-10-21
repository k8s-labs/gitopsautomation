# GitOps Automation

- GitHub Copilot Prompt
  - analyze all of the files in this repo and explain what the repo does
    - Summarized by: Claude Sonnet 4
 
> This repository enables a retail chain to safely deploy and manage applications across all their store locations using modern GitOps practices with automated, ring-based rollout strategies.

## Repository Overview

This is a GitOps automation repository designed to manage Kubernetes deployments across multiple retail store locations in Texas using a ring-based deployment strategy.

### Core Purpose

The repository automates the deployment of applications to 22+ retail store clusters located throughout central Texas, enabling controlled rollouts using a "ring" deployment pattern for safe, gradual application updates.

### Key Components

1. Cluster Configuration (config directory)

- clusters00.yaml, clusters01.yaml, clusters02.yaml: Define all retail store clusters
  - Each cluster represents a physical store location with metadata including:

- Geographic info (city, county, coordinates, population)
- Store-specific details (store number, area code, timezone)
- Ring assignment (0, 1, or 2) for deployment targeting
- Equipment metadata (oven type: ACME3000, ACME4000)

2. Application Templates (templates directory)

- Heartbeat: Health monitoring and status reporting application
- Timeclock: Employee time tracking system
- POS: Point of Sale system for retail operations
- Prometheus: Monitoring and metrics collection

3. Generated Cluster Configurations (clusters directory)

- Auto-generated store-specific Kubernetes manifests
- Each store gets its own subdirectory (e.g., tx-austin/, tx-round-rock/)
- Contains Flux GitOps listeners and application deployments
- Uses Kustomize for configuration management with base + overlay pattern

### Ring-Based Deployment Strategy

- The system uses a 3-tier ring deployment model:
  - Ring 0: Initial deployment to a few pilot stores (Austin, Cedar Park, Georgetown)
  - Ring 1: Secondary rollout to more stores (Buda, Burnet, Cedar Creek, etc.)
  - Ring 2: Final rollout to remaining stores (Bastrop, Dale, Del Valle, etc.)

### GitOps Automation Workflow

- Configuration: Store clusters defined in YAML with metadata
- Expression Evaluation: Uses expressions like /m/ring/0 to target specific store rings
- Template Processing: Combines templates with store-specific data
- Manifest Generation: Creates Kubernetes manifests for each store
- Flux Deployment: Arc-enabled GitOps (Flux) monitors repo and deploys to clusters

### Key Features

- Controlled Rollouts: Test applications on Ring 0 stores before wider deployment
- Store-Specific Configuration: Each location gets tailored Kubernetes manifests
- Automated GitOps: Flux continuously monitors and syncs configurations
- Rollback Capability: Quick rollback support for deployment issues
- Multi-Version Support: Different application versions can coexist across rings

### Technology Stack

- GitOps: Flux CD for continuous deployment
- Configuration Management: Kustomize for Kubernetes manifest templating
- Orchestration: Custom GitOps Automation (goa) CLI tool
- Infrastructure: Azure Arc-enabled Kubernetes clusters
- Monitoring: Built-in health checks and monitoring applications

## Getting Started

### Prerequisites
- Kubernetes cluster
- Flux CD installed
- kubectl configured

### Deployment

- Configure Arc enabled GitOps for your cluster

```bash

# change to your resource group
export ARC_RG=arc

# use any name from the ./clusters directory
export ARC_NAME=tx-austin

# make sure that PAT is set to a valid GitHub Personal Access Token with permissions to the repository
echo $PAT

# create flux and kustomization
az k8s-configuration flux create \
  --url https://github.com/bartr/gitopsautomation \
  --https-key $PAT \
  --cluster-name $ARC_NAME \
  --resource-group $ARC_RG \
  --cluster-type connectedClusters \
  --interval 1m \
  --kind git \
  --name gitops \
  --namespace flux-system \
  --scope cluster \
  --timeout 3m \
  --https-user gitops \
  --branch main \
  --kustomization \
      name=flux-listeners \
      path=./clusters/$ARC_NAME/flux-system/listeners \
      timeout=3m \
      sync_interval=1m \
      retry_interval=1m \
      prune=true \
      force=true

```

- Apply the cluster-specific configurations
- Monitor the deployment through Arc

## Contributing
- Create a new branch for your changes
- Make your changes
- Ensure all tests pass
- Submit a pull request
- Get approval from maintainers

## Security
- Access to the repository is restricted
- All changes require review and approval

## Support
For support, please create an issue in this repository
