# GitOps Automation

## Repository Overview

- Run the following GitHub Copilot prompt
  - analyze all of the files in this repo and explain what the repo does

## Prerequisites

- Kubernetes cluster
- kubectl configured

## Deployment

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

- Monitor the deployment through Arc

## Support

- For support, please create an issue in this repository
