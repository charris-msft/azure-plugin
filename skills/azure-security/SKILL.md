---
name: Azure Security
description: This skill should be used when the user asks about "Key Vault", "Azure secrets", "certificates", "managed identity", "RBAC", "Azure permissions", "service principal", "Azure security", or mentions Azure security and identity services. Provides best practices and MCP tool guidance for Azure security services.
---

# Azure Security Best Practices

## Overview

Azure provides comprehensive security services for identity, secrets management, and access control. This skill covers Key Vault, Managed Identities, and RBAC (Role-Based Access Control).

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools:
- `azure_keyvault_list` - List Key Vaults
- `azure_keyvault_secret_list` - List secrets in a vault
- `azure_keyvault_secret_get` - Get secret value
- `azure_keyvault_key_list` - List keys in a vault
- `azure_keyvault_certificate_list` - List certificates
- `azure_rbac_role_assignment_list` - List role assignments
- `azure_rbac_role_definition_list` - List role definitions

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup`.

## Azure Key Vault

### Overview

Key Vault provides secure storage for:
- **Secrets** - API keys, connection strings, passwords
- **Keys** - Cryptographic keys for encryption
- **Certificates** - TLS/SSL certificates

### SKUs

| SKU | Use Case | HSM Backed |
|-----|----------|------------|
| Standard | General purpose | Software |
| Premium | High security | Hardware HSM |

Use Premium for:
- Regulatory compliance requirements
- Hardware security module protection
- Financial services, healthcare

### Naming and Organization

**Vault naming:** `kv-{app}-{env}-{region}`
Example: `kv-contoso-prod-eastus`

**Organization strategies:**
- One vault per application per environment
- Separate vaults for different security boundaries
- Shared vault for common infrastructure secrets

### Access Policies vs RBAC

**Access Policies (Legacy):**
- Vault-level permissions
- All-or-nothing per operation type
- No granular secret-level control

**RBAC (Recommended):**
- Fine-grained permissions
- Secret-level access control
- Consistent with Azure-wide RBAC
- Audit trail in Activity Log

Enable RBAC permission model:
```
Set "Permission model" to "Azure role-based access control"
```

### Security Best Practices

1. **Enable soft delete and purge protection**
   - Soft delete: Recover deleted secrets (90 days)
   - Purge protection: Prevent permanent deletion

2. **Use private endpoints**
   - No public internet access
   - Access only from VNet

3. **Enable logging**
   - Diagnostic settings to Log Analytics
   - Monitor access patterns

4. **Rotate secrets regularly**
   - Set expiration dates
   - Automate rotation with Event Grid

5. **Backup critical secrets**
   - Export to secure storage
   - Cross-region replication for DR

### Secret Management Patterns

**Application configuration:**
```
Store in Key Vault:
- Database connection strings
- API keys for external services
- Encryption keys
- Certificate private keys

Reference via:
- Managed identity (preferred)
- Key Vault references in App Service
- SDK with DefaultAzureCredential
```

**Secret rotation:**
1. Create new version of secret
2. Update applications to use new version
3. Monitor for old version usage
4. Delete old version after grace period

## Managed Identities

### Types

| Type | Use Case | Lifecycle |
|------|----------|-----------|
| System-assigned | Single resource | Tied to resource |
| User-assigned | Multiple resources | Independent |

### When to Use Each

**System-assigned:**
- Simple scenarios
- One-to-one resource mapping
- Automatic cleanup on resource deletion

**User-assigned:**
- Share identity across resources
- Pre-create before resources
- Blue-green deployments

### Supported Services

Most Azure PaaS services support managed identity:
- App Service / Functions
- Virtual Machines
- AKS (workload identity)
- Container Apps
- Logic Apps
- Data Factory
- And many more...

### Best Practices

1. **Use managed identity over service principals**
   - No credentials to manage
   - Automatic rotation
   - Better security posture

2. **Apply least privilege**
   - Grant only required permissions
   - Use built-in roles when possible
   - Scope to specific resources

3. **Use DefaultAzureCredential in code**
   - Works locally and in Azure
   - Automatic credential chain
   - No code changes for deployment

### Common Patterns

**Access Key Vault:**
1. Enable managed identity on service
2. Grant Key Vault RBAC role (e.g., Key Vault Secrets User)
3. Access secrets without credentials

**Access Storage:**
1. Enable managed identity
2. Grant Storage Blob Data role
3. Access blobs with identity

## Role-Based Access Control (RBAC)

### Core Concepts

- **Security Principal** - User, group, service principal, managed identity
- **Role Definition** - Collection of permissions
- **Scope** - Where role applies (management group, subscription, resource group, resource)
- **Role Assignment** - Attaching role to principal at scope

### Built-in Roles

| Role | Permissions | Common Use |
|------|-------------|------------|
| Owner | Full access + assign roles | Admins |
| Contributor | Full access, no role assignment | Developers |
| Reader | Read-only | Auditors |
| User Access Administrator | Manage role assignments | Security |

### Service-Specific Roles

**Key Vault:**
- Key Vault Administrator - Full vault management
- Key Vault Secrets Officer - Manage secrets
- Key Vault Secrets User - Read secrets only
- Key Vault Certificates Officer - Manage certificates
- Key Vault Crypto Officer - Manage keys

**Storage:**
- Storage Blob Data Owner - Full blob access
- Storage Blob Data Contributor - Read/write blobs
- Storage Blob Data Reader - Read blobs only

### Best Practices

1. **Use built-in roles** when possible
2. **Apply least privilege** - minimum required permissions
3. **Scope narrowly** - resource level when possible
4. **Use groups** for user assignments
5. **Review assignments regularly** with Access Reviews
6. **Use PIM** (Privileged Identity Management) for sensitive roles

### Custom Roles

Create custom roles when built-in roles don't fit:
```json
{
  "Name": "Custom Role Name",
  "Description": "Description of permissions",
  "Actions": [
    "Microsoft.Storage/storageAccounts/read",
    "Microsoft.Storage/storageAccounts/blobServices/containers/read"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

## Common Operations with MCP

### Manage Key Vault

```
1. List vaults with azure_keyvault_list
2. List secrets with azure_keyvault_secret_list
3. Get secret value with azure_keyvault_secret_get
4. List keys with azure_keyvault_key_list
5. List certificates with azure_keyvault_certificate_list
```

### Check RBAC Assignments

```
1. List role assignments with azure_rbac_role_assignment_list
2. Filter by principal or scope
3. Review role definitions with azure_rbac_role_definition_list
```

## Security Checklist

### Key Vault
- [ ] Soft delete enabled
- [ ] Purge protection enabled
- [ ] RBAC permission model
- [ ] Private endpoint configured
- [ ] Diagnostic logging enabled
- [ ] Secret expiration dates set

### Identity
- [ ] Managed identity for all services
- [ ] No hardcoded credentials
- [ ] Service principals have expiring secrets
- [ ] Conditional access policies configured

### RBAC
- [ ] Least privilege applied
- [ ] Role assignments reviewed
- [ ] No Owner at subscription level (use PIM)
- [ ] Groups used for user access
- [ ] Custom roles documented

## Troubleshooting

**Access denied to Key Vault:**
1. Check RBAC role assignment
2. Verify network access (firewall/VNet)
3. Confirm managed identity is enabled
4. Check secret/key hasn't expired

**Managed identity not working:**
1. Verify identity is enabled
2. Check RBAC assignment scope
3. Ensure correct identity (system vs user-assigned)
4. Review token acquisition logs

## MCP Tool Reference

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List Key Vaults | `azure_keyvault_list` | Get all vaults |
| List secrets | `azure_keyvault_secret_list` | Get secrets in vault |
| Get secret | `azure_keyvault_secret_get` | Get secret value |
| List keys | `azure_keyvault_key_list` | Get keys in vault |
| List certificates | `azure_keyvault_certificate_list` | Get certificates |
| List role assignments | `azure_rbac_role_assignment_list` | Get RBAC assignments |
| List role definitions | `azure_rbac_role_definition_list` | Get role definitions |
