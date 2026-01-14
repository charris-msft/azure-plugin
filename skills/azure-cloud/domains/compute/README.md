# Azure Compute Services

## Services

| Service | Use When | MCP Tools | CLI |
|---------|----------|-----------|-----|
| Container Apps | Microservices, serverless containers | `azure_container_app_*` | `az containerapp` |
| App Service | Web apps, REST APIs, managed platform | `azure_appservice_*` | `az webapp` |
| Azure Functions | Event-driven, pay-per-execution | `azure_function_*` | `func`, `az functionapp` |
| AKS | Full Kubernetes control, complex architectures | `azure_aks_*` | `az aks` |
| VMs | Legacy apps, custom requirements | - | `az vm` |

## Deployment: azd vs az

**For deploying applications, prefer Azure Developer CLI (azd).**

| Tool | Best For |
|------|----------|
| **azd** | Deploying apps with infrastructure (recommended) |
| **az** | Managing existing resources, queries |

### azd Quick Start

```bash
# Initialize from template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Provision AND deploy
azd up

# Deploy code only (after initial setup)
azd deploy

# Tear down everything
azd down
```

## MCP Server (Preferred)

When Azure MCP is enabled:

- `azure_container_app_list` - List container apps
- `azure_appservice_webapp_list` - List web apps
- `azure_appservice_webapp_get` - Get app details
- `azure_function_app_list` - List function apps
- `azure_aks_cluster_list` - List AKS clusters
- `azure_aks_nodepool_list` - List node pools

**If Azure MCP is not enabled:** Run `/azure:setup` or enable via `/mcp`.

## Choosing the Right Compute

| If your app is... | Use | Why |
|-------------------|-----|-----|
| HTTP APIs, microservices | Container Apps | Serverless, auto-scale, Dapr |
| Traditional web apps | App Service | Managed platform, easy deployment |
| Event-driven functions | Azure Functions | Pay-per-execution, triggers |
| Complex K8s workloads | AKS | Full Kubernetes control |

## CLI Fallback

```bash
# Container Apps
az containerapp list --output table

# App Service
az webapp list --output table

# Functions
az functionapp list --output table

# AKS
az aks list --output table
```

## Service Details

For deep documentation on specific services:

- Container Apps deployment and scaling -> `services/container-apps.md`
- App Service plans and slots -> `services/app-service.md`
- Functions triggers and hosting plans -> `services/functions.md`
- AKS cluster configuration -> `services/aks.md`
