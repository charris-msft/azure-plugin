---
name: Azure Data Services
description: This skill should be used when the user asks about "Azure SQL", "SQL Database", "Cosmos DB", "Redis Cache", "database on Azure", "NoSQL", "document database", "caching", or mentions Azure database and data services. Provides best practices and MCP tool guidance for Azure data services.
---

# Azure Data Services Best Practices

## Overview

Azure provides multiple database services for different data models and workloads. This skill covers Azure SQL Database, Cosmos DB, and Azure Cache for Redis.

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools:
- `azure_sql_server_list` - List SQL servers
- `azure_sql_database_list` - List databases on a server
- `azure_sql_firewall_list` - List firewall rules
- `azure_cosmosdb_account_list` - List Cosmos DB accounts
- `azure_cosmosdb_database_list` - List databases in account
- `azure_cosmosdb_container_list` - List containers
- `azure_redis_cache_list` - List Redis caches

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup`.

## Choosing the Right Database

| Service | Data Model | Use Case | Scale |
|---------|-----------|----------|-------|
| Azure SQL | Relational | OLTP, structured data | Single region |
| Cosmos DB | Multi-model | Global, low latency | Multi-region |
| Redis Cache | Key-value | Caching, sessions | Regional |
| PostgreSQL | Relational | Open source preference | Single region |
| MySQL | Relational | LAMP stack | Single region |

### Decision Guide

**Choose Azure SQL when:**
- Existing SQL Server workloads
- Complex transactions (ACID)
- Familiar T-SQL skills
- Single-region deployment

**Choose Cosmos DB when:**
- Global distribution required
- Multiple data models needed
- Guaranteed low latency (<10ms)
- Massive scale (millions RPS)

**Choose Redis when:**
- Caching layer needed
- Session state management
- Real-time leaderboards
- Pub/sub messaging

## Azure SQL Database

### Service Tiers

| Tier | Use Case | vCores | Storage |
|------|----------|--------|---------|
| Basic | Dev/test | Shared | 2 GB |
| Standard | Small production | Shared | 250 GB |
| Premium | High IOPS | Dedicated | 4 TB |
| Hyperscale | Large databases | Dedicated | 100 TB |
| Serverless | Variable workloads | Auto-scale | 4 TB |

### DTU vs vCore

**DTU model:** Bundled compute/storage/IO - simpler
**vCore model:** Separate compute/storage - more control

Recommend vCore for:
- Existing SQL Server licenses (Azure Hybrid Benefit)
- Fine-grained resource control
- Reserved capacity pricing

### High Availability Options

| Option | SLA | Use Case |
|--------|-----|----------|
| Zone redundant | 99.995% | Regional HA |
| Geo-replication | RPO <5s | Disaster recovery |
| Auto-failover groups | Automatic | Multi-region HA |

### Security Best Practices

1. **Enable Azure AD authentication** - avoid SQL auth
2. **Use private endpoints** - no public internet access
3. **Enable TDE** (Transparent Data Encryption) - default on
4. **Configure auditing** to Storage or Log Analytics
5. **Use Always Encrypted** for sensitive columns
6. **Enable Advanced Threat Protection**

### Performance Optimization

1. **Index tuning** - use automatic tuning
2. **Query performance insights** - identify slow queries
3. **Elastic pools** - share resources across databases
4. **Read replicas** - offload read workloads
5. **Intelligent query processing** - automatic optimization

## Cosmos DB

### API Selection

| API | Data Model | Use Case |
|-----|-----------|----------|
| NoSQL (Core) | Document | Most scenarios, native |
| MongoDB | Document | MongoDB compatibility |
| PostgreSQL | Relational | Distributed PostgreSQL |
| Cassandra | Wide-column | Cassandra workloads |
| Gremlin | Graph | Graph relationships |
| Table | Key-value | Azure Table migration |

### Partitioning Strategy

**Partition key selection is critical:**
- High cardinality (many distinct values)
- Even distribution across partitions
- Included in most queries

Good partition keys:
- `userId` for user data
- `tenantId` for multi-tenant
- `deviceId` for IoT

Bad partition keys:
- `status` (few values)
- `timestamp` (hot partition)
- `region` (uneven distribution)

### Consistency Levels

| Level | Guarantees | Latency | Use Case |
|-------|-----------|---------|----------|
| Strong | Linearizable | Highest | Financial |
| Bounded | Bounded staleness | High | Inventory |
| Session | Session consistency | Medium | Most apps |
| Consistent Prefix | Order preserved | Low | Audit logs |
| Eventual | No guarantees | Lowest | Recommendations |

Default to **Session** consistency for most applications.

### Request Units (RUs)

RU = normalized cost of operations
- Point read (1KB): 1 RU
- Write (1KB): ~5 RU
- Query: varies by complexity

**Provisioned throughput:** Set RU/s, consistent cost
**Autoscale:** 10-100% of max, automatic
**Serverless:** Pay per request, variable workloads

### Cost Optimization

1. **Use autoscale** instead of manual provisioning
2. **Optimize partition keys** to avoid cross-partition queries
3. **Enable TTL** for automatic data expiration
4. **Use analytical store** for analytics (separate from transactional)
5. **Reserved capacity** for 1-3 year commitment

## Azure Cache for Redis

### Tiers

| Tier | Features | Use Case |
|------|----------|----------|
| Basic | Single node, no SLA | Dev/test |
| Standard | Replicated, 99.9% SLA | Production |
| Premium | Clustering, VNet, persistence | High scale |
| Enterprise | Redis modules, 99.99% SLA | Enterprise |

### Caching Patterns

**Cache-Aside:**
1. Check cache first
2. If miss, query database
3. Store result in cache
4. Return data

**Write-Through:**
1. Write to cache
2. Cache writes to database
3. Ensures consistency

**Write-Behind:**
1. Write to cache
2. Async write to database
3. Better performance, eventual consistency

### Best Practices

1. **Set appropriate TTLs** - avoid stale data
2. **Use connection pooling** - reuse connections
3. **Enable clustering** for scale-out
4. **Configure eviction policy** (allkeys-lru recommended)
5. **Monitor memory usage** - avoid evictions

### Common Use Cases

- **Session state** - distributed session storage
- **Output caching** - page/API response caching
- **Distributed locking** - coordination across instances
- **Rate limiting** - request throttling
- **Leaderboards** - sorted sets for rankings

## Common Operations with MCP

### Query SQL Databases

```
1. List servers with azure_sql_server_list
2. List databases with azure_sql_database_list
3. Check firewall rules with azure_sql_firewall_list
```

### Explore Cosmos DB

```
1. List accounts with azure_cosmosdb_account_list
2. List databases with azure_cosmosdb_database_list
3. List containers with azure_cosmosdb_container_list
```

### Check Redis Caches

```
Use azure_redis_cache_list to view all caches.
Check tier, size, and configuration.
```

## Migration Strategies

### SQL Server to Azure SQL

1. **Assessment** - Use Data Migration Assistant
2. **Schema migration** - Generate scripts or use tools
3. **Data migration** - DMS, bacpac, or replication
4. **Cutover** - Switch connection strings
5. **Validation** - Verify data integrity

### MongoDB to Cosmos DB

1. Use Azure Database Migration Service
2. Pre-migration assessment
3. Offline or online migration
4. Validate and optimize partition keys

## Security Checklist

- [ ] Private endpoints enabled
- [ ] Managed identity for connections
- [ ] Encryption at rest (TDE/CMK)
- [ ] Encryption in transit (TLS 1.2+)
- [ ] Firewall rules configured
- [ ] Auditing enabled
- [ ] Threat detection enabled

## MCP Tool Reference

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List SQL servers | `azure_sql_server_list` | Get SQL servers |
| List SQL databases | `azure_sql_database_list` | Get databases |
| List firewall rules | `azure_sql_firewall_list` | Get SQL firewall rules |
| List Cosmos accounts | `azure_cosmosdb_account_list` | Get Cosmos DB accounts |
| List Cosmos databases | `azure_cosmosdb_database_list` | Get databases |
| List Cosmos containers | `azure_cosmosdb_container_list` | Get containers |
| List Redis caches | `azure_redis_cache_list` | Get Redis caches |
