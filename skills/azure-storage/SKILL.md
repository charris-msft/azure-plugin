---
name: Azure Storage
description: This skill should be used when the user asks about "Azure storage", "blob storage", "storage accounts", "containers", "upload to Azure", "download from blob", "list blobs", "storage queues", "file shares", or mentions Azure Storage operations. Provides best practices and MCP tool guidance for Azure Storage services.
---

# Azure Storage Best Practices

## Overview

Azure Storage provides massively scalable cloud storage for data objects, file systems, messaging, and NoSQL data. This skill covers Blob storage, Queue storage, Table storage, and Azure Files.

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools for storage operations:
- `azure_storage_account_list` - List storage accounts in subscription
- `azure_storage_container_list` - List containers in a storage account
- `azure_storage_blob_list` - List blobs in a container
- `azure_storage_blob_get` - Download blob content
- `azure_storage_blob_put` - Upload blob content

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup` for guided configuration.

## Storage Account Best Practices

### Naming Conventions

Storage account names must be:
- Between 3-24 characters
- Lowercase letters and numbers only
- Globally unique across all Azure

Recommended pattern: `{company}{environment}{purpose}{region}`
Example: `contosoprodsaborageseus`

### Performance Tiers

| Tier | Use Case | IOPS | Latency |
|------|----------|------|---------|
| Standard | General purpose, backup, archive | Up to 20,000 | Milliseconds |
| Premium | High-performance, databases | Up to 100,000 | Sub-millisecond |

Select Premium for:
- Production databases
- High-frequency transaction processing
- Real-time analytics

### Redundancy Options

| Type | Description | Durability |
|------|-------------|------------|
| LRS | Local redundant (3 copies in one datacenter) | 11 nines |
| ZRS | Zone redundant (3 copies across zones) | 12 nines |
| GRS | Geo-redundant (6 copies, 2 regions) | 16 nines |
| GZRS | Geo-zone redundant (best durability) | 16 nines |

Choose based on:
- **LRS**: Dev/test, easily recreatable data
- **ZRS**: High availability within region
- **GRS/GZRS**: Business critical, disaster recovery

## Blob Storage

### Access Tiers

| Tier | Access Frequency | Storage Cost | Access Cost |
|------|-----------------|--------------|-------------|
| Hot | Frequent | Highest | Lowest |
| Cool | Infrequent (30+ days) | Lower | Higher |
| Cold | Rare (90+ days) | Lower still | Higher still |
| Archive | Rarely (180+ days) | Lowest | Highest + rehydration |

Implement lifecycle management policies to automatically transition blobs between tiers.

### Container Organization

Structure containers by:
- Application or service
- Data classification (public, private, confidential)
- Retention requirements

Example structure:
```
/public-assets          # CDN-served static content
/user-uploads           # User-generated content
/application-logs       # Log files with retention policy
/backups                # Database backups, archive tier
```

### Security Best Practices

1. **Use managed identities** instead of storage keys when possible
2. **Enable soft delete** for blob and container recovery
3. **Configure CORS** only for required origins
4. **Use private endpoints** for VNet integration
5. **Enable versioning** for critical data
6. **Disable public blob access** unless specifically required

### Shared Access Signatures (SAS)

Generate SAS tokens with minimal permissions:
- Specify exact permissions needed (read, write, delete, list)
- Set short expiration times
- Use stored access policies for revocability
- Prefer user delegation SAS over account SAS

## Queue Storage

### Design Patterns

**At-least-once processing**: Messages may be delivered multiple times. Design handlers to be idempotent.

**Visibility timeout**: Set based on expected processing time plus buffer. Default is 30 seconds.

**Poison message handling**: After N failed attempts, move message to dead-letter queue.

### Best Practices

1. Keep messages small (<64KB), store large payloads in blob storage
2. Use batch operations for high throughput
3. Implement exponential backoff for retries
4. Monitor queue depth for scaling triggers

## Azure Files

### Use Cases

- Lift-and-shift applications requiring SMB file shares
- Shared configuration across VMs
- Development environments needing shared storage
- Container persistent volumes

### Performance Optimization

1. **Use Premium tier** for latency-sensitive workloads
2. **Enable large file shares** for up to 100 TiB capacity
3. **Consider Azure File Sync** for hybrid scenarios
4. **Mount with SMB 3.0** for encryption in transit

## Common Operations with MCP

### List Storage Accounts

```
Use azure_storage_account_list to retrieve all storage accounts.
Filter by resource group if needed.
```

### Manage Blobs

```
1. List containers with azure_storage_container_list
2. List blobs with azure_storage_blob_list
3. Download with azure_storage_blob_get
4. Upload with azure_storage_blob_put
```

### Troubleshooting

**Access denied errors:**
- Verify managed identity has correct RBAC role
- Check network rules (firewall, VNet)
- Ensure SAS token hasn't expired

**Throttling:**
- Implement retry with exponential backoff
- Consider partitioning data across accounts
- Upgrade to Premium tier if hitting limits

## Cost Optimization

1. **Use lifecycle management** to move data to cheaper tiers
2. **Enable soft delete cautiously** - it increases storage costs
3. **Delete unused snapshots** and old versions
4. **Right-size redundancy** - don't use GRS for dev environments
5. **Monitor with Storage Analytics** to identify optimization opportunities

## Integration Patterns

### Static Website Hosting

Enable static website hosting for SPAs:
1. Enable in storage account settings
2. Upload index.html and error.html
3. Configure custom domain with CDN

### Event-Driven Processing

Connect Blob storage to Event Grid:
- Trigger Functions on blob create/delete
- Process uploaded files automatically
- Implement ETL pipelines

### Backup Strategy

Implement 3-2-1 backup rule:
- 3 copies of data
- 2 different storage types
- 1 offsite (different region with GRS)

## MCP Tool Reference

Ensure Azure MCP server is enabled to use these operations. Check status with `/azure:status` or enable via `/mcp`.

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List accounts | `azure_storage_account_list` | Get all storage accounts |
| List containers | `azure_storage_container_list` | Get containers in account |
| List blobs | `azure_storage_blob_list` | Get blobs in container |
| Get blob | `azure_storage_blob_get` | Download blob content |
| Put blob | `azure_storage_blob_put` | Upload blob content |
