---
name: auth
description: Help with Azure authentication issues and credential management
argument-hint: "[issue-description]"
allowed-tools: ["Bash", "Read"]
---

# Azure Authentication Help Command

Diagnose and resolve Azure authentication issues.

## Diagnostic Steps

### 1. Check Current Authentication Status

```bash
az account show 2>&1
```

Analyze the output:
- If successful: Show current account and subscription
- If "Please run 'az login'": User needs to authenticate
- If error: Diagnose specific issue

### 2. Common Authentication Scenarios

#### Interactive Login (Browser)

For users with browser access:
```bash
az login
```

#### Device Code Login (No Browser)

For headless or remote environments:
```bash
az login --use-device-code
```

Instruct user to:
1. Open https://microsoft.com/devicelogin
2. Enter the displayed code
3. Complete authentication in browser

#### Service Principal Login

For automated/CI scenarios:
```bash
az login --service-principal \
  --username <app-id> \
  --password <client-secret> \
  --tenant <tenant-id>
```

Required information:
- Application (client) ID
- Client secret or certificate
- Tenant ID

#### Managed Identity Login

For Azure-hosted environments (VMs, App Service, etc.):
```bash
az login --identity
```

For user-assigned managed identity:
```bash
az login --identity --username <managed-identity-client-id>
```

### 3. Token Management

#### Check Token Expiration

```bash
az account get-access-token --query "expiresOn" -o tsv
```

If token is expired or expiring soon:
```bash
az account get-access-token --resource https://management.azure.com/
```

#### Clear Cached Credentials

If experiencing stale credential issues:
```bash
az account clear
az login
```

### 4. Subscription Management

#### List All Subscriptions

```bash
az account list --output table
```

#### Switch Subscription

```bash
az account set --subscription "<name-or-id>"
```

#### Verify Current Subscription

```bash
az account show --query "{Name:name, ID:id, State:state}" -o table
```

## Common Issues and Solutions

### Issue: "AADSTS50076" - MFA Required

**Cause:** Multi-factor authentication is required but not completed.

**Solution:**
```bash
az login --use-device-code
```
Complete MFA in the browser.

### Issue: "AADSTS700016" - Application Not Found

**Cause:** Service principal doesn't exist or wrong tenant.

**Solution:**
1. Verify app ID is correct
2. Confirm app exists in the tenant
3. Check tenant ID matches

### Issue: "AADSTS7000215" - Invalid Client Secret

**Cause:** Client secret is wrong or expired.

**Solution:**
1. Generate new client secret in Azure Portal
2. Update credential in login command
3. Note: Secrets can expire

### Issue: "No subscriptions found"

**Cause:** Account has no Azure subscriptions or lacks access.

**Solution:**
1. Verify account should have subscription access
2. Check Azure Portal for subscription visibility
3. Contact Azure administrator for access

### Issue: Token Refresh Failures

**Cause:** Cached tokens are corrupted or expired.

**Solution:**
```bash
az account clear
az cache purge
az login
```

## Environment Variables

For non-interactive scenarios, set these environment variables:

```bash
# Service Principal
export AZURE_TENANT_ID="<tenant-id>"
export AZURE_CLIENT_ID="<client-id>"
export AZURE_CLIENT_SECRET="<client-secret>"

# For Azure MCP Server
# These are read automatically if set
```

## Security Best Practices

1. **Never hardcode credentials** in scripts or code
2. **Use managed identity** when running in Azure
3. **Rotate secrets regularly** (recommended: 90 days)
4. **Use certificate auth** over secrets when possible
5. **Limit service principal permissions** to least privilege
6. **Store secrets in Key Vault** for applications

## Output

After diagnosing, provide:
- Current authentication status
- Identified issues (if any)
- Recommended actions
- Commands to resolve issues
