---
name: status
description: Check Azure MCP server status and available tools
argument-hint: ""
allowed-tools: ["Bash", "Read", "mcp__azure__*"]
---

# Azure Status Command

Check the status of Azure MCP server and display available capabilities.

## Status Checks

### 1. Azure CLI Status

Check Azure CLI installation and authentication:

```bash
# Version
az version --output json 2>&1

# Current account
az account show --query "{Name:name, ID:id, TenantId:tenantId}" -o json 2>&1
```

Report:
- CLI version
- Logged in user/service principal
- Current subscription
- Tenant ID

### 2. Azure MCP Server Status

Attempt to use an MCP tool to verify the server is running:

Use `azure_subscription_list` to:
1. Confirm MCP server is responsive
2. Verify authentication is working
3. Get subscription count

If MCP tools are not available:
- Inform user the Azure MCP server is not enabled
- Direct them to `/mcp` to enable it
- Or run `/azure:setup` for guided configuration

### 3. Available MCP Tools

List the major tool categories available:

| Category | Tools | Description |
|----------|-------|-------------|
| Subscription | `azure_subscription_list` | List subscriptions |
| Resource Groups | `azure_resource_group_list` | List resource groups |
| Storage | `azure_storage_*` | Storage operations |
| Compute | `azure_appservice_*`, `azure_aks_*` | Compute services |
| Data | `azure_sql_*`, `azure_cosmosdb_*` | Database services |
| Security | `azure_keyvault_*`, `azure_rbac_*` | Security services |
| AI | `azure_search_*`, `azure_speech_*` | AI services |
| CLI | `azure_cli_generate` | Generate CLI commands |

### 4. Quick Health Check

If MCP is enabled, perform quick validation:

1. **Subscription access**: List subscriptions
2. **Resource access**: List resource groups (if subscription selected)
3. **Tool availability**: Confirm tools respond

## Output Format

Display a comprehensive status report:

```
Azure MCP Status
================

Azure CLI
  Version: 2.x.x
  Status: ✓ Installed

Authentication
  User: user@contoso.com
  Type: Interactive
  Status: ✓ Authenticated

Subscription
  Name: Contoso Production
  ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  State: Enabled

Azure MCP Server
  Status: ✓ Running
  Tools Available: 40+

Quick Test Results
  Subscriptions: ✓ 3 found
  Resource Groups: ✓ 15 found

Ready to use Azure tools!
```

## Error States

### MCP Server Not Enabled

```
Azure MCP Server
  Status: ✗ Not enabled

To enable:
  1. Run /mcp
  2. Enable the "azure" server
  3. Run /azure:status again
```

### Authentication Required

```
Authentication
  Status: ✗ Not authenticated

To authenticate:
  Run /azure:auth or: az login
```

### No Subscription Access

```
Subscription
  Status: ✗ No subscriptions found

Possible causes:
  - Account has no Azure subscriptions
  - Insufficient permissions
  - Wrong tenant selected
```

## Tips

After displaying status, offer helpful suggestions:

- If everything is working: "Try asking about your Azure resources!"
- If MCP not enabled: "Enable Azure MCP via /mcp to use Azure tools"
- If not authenticated: "Run /azure:auth to set up authentication"
- If no subscriptions: "Contact your Azure administrator for access"
