---
name: Azure Infrastructure
description: This skill should be used when the user asks about "Azure CLI", "azd", "Azure Developer CLI", "resource groups", "subscriptions", "ARM templates", "Bicep", "Azure deployment", "infrastructure as code", "Azure resources", "deploy to Azure", or mentions Azure infrastructure and deployment services. Provides best practices for azd (preferred for app deployments) and az CLI for resource management.
---

# Azure Infrastructure Best Practices

## Overview

Azure infrastructure management encompasses resource organization, deployment automation, and governance. This skill covers Azure CLI, Resource Groups, Subscriptions, and Infrastructure as Code (ARM/Bicep).

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools:
- `azure_subscription_list` - List subscriptions
- `azure_resource_group_list` - List resource groups
- `azure_resource_list` - List resources
- `azure_deployment_list` - List deployments
- `azure_deployment_get` - Get deployment details
- `azure_cli_generate` - Generate Azure CLI commands
- `azure_quota_list` - List service quotas

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup`.

## Resource Organization

### Subscription Strategy

**Patterns:**

| Pattern | Use Case | Complexity |
|---------|----------|------------|
| Single subscription | Small org, single project | Low |
| Environment-based | Dev/Test/Prod isolation | Medium |
| Workload-based | Application isolation | Medium |
| Business unit | Org structure alignment | High |

**Recommendations:**
1. Separate production from non-production
2. Use management groups for policy inheritance
3. Implement subscription quotas and budgets
4. Apply consistent tagging strategy

### Management Group Hierarchy

```
Root Management Group
├── Platform
│   ├── Identity
│   ├── Management
│   └── Connectivity
├── Landing Zones
│   ├── Production
│   │   ├── Prod-Sub-1
│   │   └── Prod-Sub-2
│   └── Non-Production
│       ├── Dev-Sub
│       └── Test-Sub
└── Sandbox
    └── Sandbox-Sub
```

### Resource Group Design

**Naming convention:** `rg-{workload}-{environment}-{region}-{instance}`
Example: `rg-contoso-prod-eastus-001`

**Organization strategies:**
- By application lifecycle (deploy/delete together)
- By resource type (all databases together)
- By environment (dev, test, prod)
- By billing/cost center

**Best practices:**
1. Group resources with same lifecycle
2. Keep groups focused (not too large)
3. Apply consistent tags
4. Use resource locks for critical resources
5. Consider RBAC scope when grouping

### Tagging Strategy

**Required tags:**
| Tag | Purpose | Example |
|-----|---------|---------|
| Environment | Deployment stage | prod, dev, test |
| Owner | Contact person | team-platform@contoso.com |
| CostCenter | Billing attribution | CC-12345 |
| Project | Business project | project-alpha |

**Optional tags:**
| Tag | Purpose | Example |
|-----|---------|---------|
| Application | Application name | contoso-web |
| DataClassification | Data sensitivity | confidential |
| CreatedBy | Automation source | terraform |
| ExpirationDate | Cleanup date | 2024-12-31 |

## Deployment Tools: azd vs az

**For application deployments, prefer Azure Developer CLI (azd).**

| Tool | Use For | Approach |
|------|---------|----------|
| **azd** | Deploying applications | Application-centric, infrastructure + code |
| **az** | Managing resources, queries | Resource-centric, imperative commands |
| **Bicep** | Infrastructure only | Declarative templates |

### Azure Developer CLI (azd)

**Recommended for deploying applications to Azure.**

```bash
# Check if installed
azd version

# Install (if needed)
# Windows: winget install Microsoft.Azd
# macOS: brew install azd
# Linux: curl -fsSL https://aka.ms/install-azd.sh | bash
```

#### azd Workflow

```bash
# Start from a template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Or initialize existing project
azd init

# Provision infrastructure AND deploy code
azd up

# Subsequent code deployments
azd deploy

# View what's deployed
azd show

# Clean up everything
azd down
```

#### azd Templates

Browse templates at: https://azure.github.io/awesome-azd/

**Popular templates:**
| Template | Stack | Target |
|----------|-------|--------|
| `todo-nodejs-mongo-aca` | Node.js + MongoDB | Container Apps |
| `todo-python-mongo-aca` | Python + MongoDB | Container Apps |
| `todo-csharp-sql-aca` | .NET + SQL | Container Apps |
| `todo-python-mongo-swa-func` | Python + Functions | Static Web Apps |

#### azd vs az Decision Guide

**Use azd when:**
- Deploying a new application
- Want infrastructure + code together
- Starting from a template
- Need consistent dev/test/prod environments
- Deploying to Container Apps, App Service, Functions

**Use az/Bicep when:**
- Managing existing resources
- Infrastructure-only changes
- Complex resource queries
- One-off modifications
- Resources not covered by azd templates

## Azure CLI (az)

**Best for: Resource management, queries, scripting**

### Common Commands

**Authentication:**
```bash
az login                    # Interactive login
az login --service-principal  # Service principal
az account set -s <sub-id>  # Set subscription
az account show             # Current context
```

**Resource management:**
```bash
az group list               # List resource groups
az resource list -g <rg>    # List resources in group
az resource show --ids <id> # Get resource details
```

**Deployment:**
```bash
az deployment group create  # Deploy to resource group
az deployment sub create    # Deploy to subscription
az deployment what-if       # Preview changes
```

### Using MCP CLI Generation

The `azure_cli_generate` MCP tool can generate Azure CLI commands:
```
Request: "Create a command to list all storage accounts"
Result: "az storage account list --output table"
```

## Infrastructure as Code

### ARM Templates vs Bicep

| Feature | ARM Templates | Bicep |
|---------|--------------|-------|
| Syntax | JSON | DSL (simpler) |
| Learning curve | Steeper | Easier |
| Modularity | Linked templates | Native modules |
| Intellisense | Limited | Excellent |
| Decompilation | N/A | From ARM |

**Recommendation:** Use Bicep for new projects

### Bicep Best Practices

**File structure:**
```
infra/
├── main.bicep           # Entry point
├── modules/
│   ├── storage.bicep
│   ├── networking.bicep
│   └── compute.bicep
├── parameters/
│   ├── dev.bicepparam
│   ├── test.bicepparam
│   └── prod.bicepparam
└── bicepconfig.json     # Linting rules
```

**Naming conventions:**
```bicep
// Resource naming
var storageAccountName = 'st${workload}${environment}${uniqueString(resourceGroup().id)}'

// Use camelCase for variables and parameters
param environmentName string
var resourceGroupLocation = resourceGroup().location
```

**Module pattern:**
```bicep
// main.bicep
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    storageAccountName: storageAccountName
  }
}
```

### Deployment Strategies

**What-If:**
Always preview changes before deployment:
```bash
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters @parameters/prod.bicepparam
```

**Incremental vs Complete:**
- **Incremental** (default): Add/update resources, leave others
- **Complete**: Delete resources not in template

Use incremental for production, complete only for clean environments.

**Deployment Stacks:**
Manage resources as a unit:
```bash
az stack group create \
  --name myStack \
  --resource-group rg-myapp \
  --template-file main.bicep \
  --deny-settings-mode none
```

## Governance

### Azure Policy

**Common policies:**
- Require tags on resources
- Restrict allowed locations
- Enforce naming conventions
- Require encryption
- Deny public endpoints

**Policy assignment scopes:**
- Management group (inherited)
- Subscription
- Resource group

### Resource Locks

| Lock Type | Effect |
|-----------|--------|
| ReadOnly | No modifications or deletions |
| CanNotDelete | Can modify, cannot delete |

Apply to critical resources:
- Production databases
- Networking infrastructure
- Identity resources

### Budgets and Cost Management

1. Set budgets at subscription level
2. Configure alerts at 50%, 75%, 100%
3. Use cost analysis for investigation
4. Implement resource cleanup automation
5. Review Azure Advisor recommendations

## Common Operations with MCP

### Explore Resources

```
1. List subscriptions with azure_subscription_list
2. Set context to subscription
3. List resource groups with azure_resource_group_list
4. List resources with azure_resource_list
```

### Manage Deployments

```
1. List deployments with azure_deployment_list
2. Get deployment details with azure_deployment_get
3. Review deployment status and outputs
```

### Check Quotas

```
Use azure_quota_list to check service limits.
Request increases before hitting limits.
```

### Generate CLI Commands

```
Use azure_cli_generate for complex commands.
Verify generated commands before execution.
```

## Landing Zone Patterns

### Application Landing Zone

Essential components:
- Virtual Network (spoke)
- Network Security Groups
- Route tables (if hub-spoke)
- Key Vault for secrets
- Storage account for diagnostics
- Log Analytics workspace (or connect to central)

### Data Landing Zone

Additional components:
- Azure SQL or Cosmos DB
- Data Lake Storage
- Data Factory
- Synapse Analytics (optional)
- Private endpoints

## Networking Essentials

### Hub-Spoke Topology

```
Hub VNet
├── Azure Firewall
├── VPN/ExpressRoute Gateway
├── Bastion Host
└── Central services

Spoke VNets (peered to hub)
├── Application Spoke
├── Data Spoke
└── Management Spoke
```

### Network Security

1. **NSGs** - Layer 4 filtering
2. **Azure Firewall** - Layer 7 filtering
3. **Private Endpoints** - Private PaaS access
4. **Service Endpoints** - Optimized PaaS routing
5. **DDoS Protection** - Attack mitigation

## Disaster Recovery

### Backup Strategy

| Resource Type | Backup Method |
|--------------|---------------|
| VMs | Azure Backup |
| SQL Database | Automated backups |
| Cosmos DB | Continuous backup |
| Storage | Versioning + GRS |
| App Service | Backup feature |

### Cross-Region Replication

1. Geo-redundant storage (GRS)
2. SQL geo-replication
3. Cosmos DB multi-region
4. Traffic Manager for failover
5. Azure Site Recovery for VMs

## MCP Tool Reference

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List subscriptions | `azure_subscription_list` | Get all subscriptions |
| List resource groups | `azure_resource_group_list` | Get resource groups |
| List resources | `azure_resource_list` | Get resources |
| List deployments | `azure_deployment_list` | Get deployments |
| Get deployment | `azure_deployment_get` | Get deployment details |
| Generate CLI | `azure_cli_generate` | Generate Azure CLI |
| List quotas | `azure_quota_list` | Get service quotas |

## Quick Reference

### Resource Naming

| Resource | Pattern | Example |
|----------|---------|---------|
| Resource Group | rg-{app}-{env}-{region} | rg-contoso-prod-eastus |
| Storage Account | st{app}{env}{unique} | stcontosoprod001 |
| Virtual Network | vnet-{app}-{env}-{region} | vnet-contoso-prod-eastus |
| Subnet | snet-{purpose} | snet-frontend |
| Key Vault | kv-{app}-{env} | kv-contoso-prod |
| App Service | app-{app}-{env} | app-contoso-prod |

### Essential Commands

**azd (for application deployments):**
```bash
azd init --template <template-name>   # Initialize from template
azd up                                 # Provision + deploy
azd deploy                             # Deploy code only
azd down                               # Tear down everything
```

**az (for resource management):**
```bash
# List and set subscription
az account list --output table
az account set --subscription "Sub Name"

# Resource groups
az group list --output table
az group create -n rg-name -l eastus

# Deployments (infrastructure only)
az deployment group create -g rg-name -f main.bicep
az deployment group what-if -g rg-name -f main.bicep
```
