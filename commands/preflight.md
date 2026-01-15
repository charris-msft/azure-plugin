---
name: preflight
description: Run pre-deployment checks before deploying to Azure. Validates tools, authentication, quotas, and prerequisites.
---

# Azure Pre-flight Checks

Run these checks BEFORE any Azure deployment to avoid mid-deployment failures.

## Check Sequence

Execute these checks in order and report all issues before proceeding:

### 1. Tool Installation Check

```bash
# Check Azure CLI
az version 2>/dev/null || echo "MISSING: Azure CLI (az)"

# Check Azure Developer CLI
azd version 2>/dev/null || echo "MISSING: Azure Developer CLI (azd)"

# Check Docker (if container deployment)
docker version 2>/dev/null || echo "MISSING: Docker"

# Check func (if Functions deployment)
func --version 2>/dev/null || echo "MISSING: Azure Functions Core Tools"
```

**If any tool is missing:**
- Use `azure__extension_cli_install` MCP tool to get installation instructions
- Or offer to run installation commands directly:
  - Windows: `winget install Microsoft.AzureCLI`, `winget install Microsoft.Azd`
  - macOS: `brew install azure-cli`, `brew tap azure/azd && brew install azd`

### 2. Authentication Check

```bash
# Check Azure CLI login
az account show --query "{Name:name, State:state, IsDefault:isDefault}" -o table 2>/dev/null || echo "NOT AUTHENTICATED: Run 'az login'"

# Check current subscription
az account show --query "id" -o tsv

# Check azd auth (uses az credentials)
azd auth login --check-status 2>/dev/null || echo "AZD: Will use Azure CLI credentials"
```

**If not authenticated:**
- Offer device code auth for remote/headless: `az login --use-device-code`
- Set longer timeout if auth fails: recommend user run `az login` in separate terminal

### 3. Subscription Quota Check

```bash
# Get subscription ID
SUBSCRIPTION=$(az account show --query id -o tsv)

# Check compute quotas (for Container Apps, AKS, VMs)
az quota usage list --scope "/subscriptions/$SUBSCRIPTION/providers/Microsoft.ContainerInstance/locations/eastus" 2>/dev/null || echo "Cannot check quotas - may need to verify manually"

# List current resource usage
az resource list --query "length(@)" -o tsv
```

**Use MCP tools for detailed quota check:**
```
Use azure__quota tools to check:
- Container instance limits
- App Service plan limits
- Storage account limits
- Cosmos DB account limits
```

### 4. Docker Status Check (for container deployments)

```bash
# Check if Docker daemon is running
docker info >/dev/null 2>&1 || echo "Docker daemon not running"

# If not running, offer to start:
# Windows: Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
# macOS: open -a Docker
# Linux: sudo systemctl start docker
```

### 5. Resource Name Availability

```bash
# Check ACR name availability
az acr check-name --name PROPOSED_NAME --query "nameAvailable"

# Check storage account name
az storage account check-name --name PROPOSED_NAME --query "nameAvailable"

# Check web app name
az webapp list --query "[?name=='PROPOSED_NAME']" -o tsv && echo "Name taken" || echo "Name available"
```

## Pre-flight Report Template

Generate a report like this before deployment:

```
=== Azure Pre-flight Check ===

Tools:
  ✓ Azure CLI (az): 2.55.0
  ✓ Azure Developer CLI (azd): 1.5.0
  ✓ Docker: 24.0.7
  ✗ Functions Core Tools: NOT INSTALLED (needed for Functions deployment)

Authentication:
  ✓ Logged in as: user@domain.com
  ✓ Subscription: My-Subscription (xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
  ✓ Tenant: My-Tenant

Quotas (eastus):
  ✓ Container Apps environments: 2/5 used
  ✓ Container App replicas: 10/100 used
  ⚠ Storage accounts: 245/250 used (NEAR LIMIT)

Docker:
  ✓ Docker daemon running
  ✓ Can pull from mcr.microsoft.com

Recommendations:
  1. Install Functions Core Tools: winget install Microsoft.Azure.FunctionsCoreTools
  2. Consider different region - storage accounts near quota in eastus

Ready to deploy: YES (with warnings)
```

## Common Issues and Fixes

| Issue | Fix |
|-------|-----|
| `az: command not found` | `winget install Microsoft.AzureCLI` |
| `azd: command not found` | `winget install Microsoft.Azd` |
| Device code auth timeout | Run `az login` in separate terminal |
| Docker not running | Start Docker Desktop, wait 30-60 seconds |
| Quota exceeded | Choose different region or request increase |
| Name already taken | Append random suffix or use different name |

## Fallback Service Recommendations

If primary service has quota issues:

| Primary | Fallback | When to Use |
|---------|----------|-------------|
| Container Apps | App Service | Container Apps quota exhausted |
| AKS | Container Apps | Simpler deployment needed |
| Cosmos DB | Azure SQL | Relational data patterns |
| Premium Storage | Standard Storage | Cost optimization |

## After Pre-flight

Once all checks pass:
1. Run `azd up` for full deployment (preferred)
2. Or `azd provision` then `azd deploy` for staged approach
3. Use `azd down --force --purge` to clean up test deployments
