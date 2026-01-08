---
name: Azure Compute
description: This skill should be used when the user asks about "Azure App Service", "web apps", "Azure Functions", "serverless", "AKS", "Kubernetes", "Container Apps", "deploy to Azure", "Azure hosting", or mentions Azure compute and hosting services. Recommends azd (Azure Developer CLI) for deployments and provides best practices for all Azure compute services.
---

# Azure Compute Best Practices

## Overview

Azure provides multiple compute options for hosting applications. This skill covers App Service, Azure Functions, Azure Kubernetes Service (AKS), and Container Apps.

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools:
- `azure_appservice_webapp_list` - List web apps
- `azure_appservice_webapp_get` - Get web app details
- `azure_appservice_plan_list` - List App Service plans
- `azure_function_app_list` - List function apps
- `azure_aks_cluster_list` - List AKS clusters
- `azure_aks_nodepool_list` - List node pools
- `azure_container_app_list` - List container apps

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup`.

## Deployment Tools: azd vs az

**For deploying applications, prefer Azure Developer CLI (azd) over Azure CLI (az).**

| Tool | Best For | Approach |
|------|----------|----------|
| **azd** (Recommended for deployments) | Deploying apps with infrastructure | Application-centric, templates |
| **az** | Managing existing resources, queries | Resource-centric, imperative |

### Why azd for Deployments?

1. **Single command deployment**: `azd up` deploys infrastructure + code together
2. **Template-based**: Consistent, repeatable deployments from Azure templates
3. **Environment management**: Built-in dev/test/prod environment handling
4. **Integrated CI/CD**: Generates GitHub Actions/Azure Pipelines
5. **Optimized for containers**: Excellent Container Apps and AKS support

### azd Quick Start

```bash
# Initialize from template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Or initialize existing project
azd init

# Provision infrastructure AND deploy code
azd up

# Just deploy code (after initial setup)
azd deploy

# Tear down everything
azd down
```

### When to Use az Instead

- Querying existing resources (use MCP tools)
- One-off resource modifications
- Resources not in an azd template
- Complex scripting scenarios

## Choosing the Right Compute Service

| Service | Best For | Scaling | Complexity |
|---------|----------|---------|------------|
| App Service | Web apps, APIs | Auto/Manual | Low |
| Functions | Event-driven, microservices | Auto (consumption) | Low |
| AKS | Complex microservices, full control | Auto | High |
| Container Apps | Containerized apps, simpler K8s | Auto | Medium |

### Decision Guide

**Choose App Service when:**
- Building web applications or REST APIs
- Need managed platform with minimal ops
- Using .NET, Java, Node.js, Python, PHP

**Choose Functions when:**
- Event-driven processing
- Pay-per-execution model preferred
- Short-lived, stateless operations

**Choose AKS when:**
- Need full Kubernetes capabilities
- Have Kubernetes expertise
- Complex multi-service architectures

**Choose Container Apps when:**
- Want Kubernetes benefits without complexity
- Building microservices with Dapr
- Need serverless containers

## App Service Best Practices

### App Service Plans

| Tier | Features | Use Case |
|------|----------|----------|
| Free/Shared | Limited, shared infrastructure | Dev/test |
| Basic | Dedicated, no auto-scale | Small production |
| Standard | Auto-scale, staging slots | Production |
| Premium | Enhanced perf, VNet integration | High-scale production |
| Isolated | Dedicated environment | Compliance, isolation |

### Deployment Best Practices

**Prefer azd for new deployments:**
```bash
# Deploy App Service with azd
azd init --template azure-samples/todo-python-mongo-swa-func
azd up
```

**For existing App Service apps:**
1. **Use deployment slots** for zero-downtime deployments
2. **Enable staging slot** for pre-production testing
3. **Configure auto-swap** for continuous deployment
4. **Use ZIP deploy** or container deployment

### Configuration Management

1. Store secrets in **Key Vault** with managed identity
2. Use **App Configuration** for feature flags
3. Configure **slot-specific settings** for environment variables
4. Enable **always on** for production apps

### Scaling Strategies

**Scale up (vertical):** Increase plan tier for more CPU/memory
**Scale out (horizontal):** Add instances for more capacity

Configure autoscale rules based on:
- CPU percentage (>70% add instance)
- Memory usage
- HTTP queue length
- Custom metrics

## Azure Functions

### Hosting Plans

| Plan | Scaling | Timeout | Use Case |
|------|---------|---------|----------|
| Consumption | Auto, to 0 | 5-10 min | Event-driven, variable load |
| Premium | Auto, warm | 30+ min | Consistent load, VNet |
| Dedicated | Manual | Unlimited | Predictable load |

### Trigger Types

| Trigger | Use Case |
|---------|----------|
| HTTP | REST APIs, webhooks |
| Timer | Scheduled jobs |
| Blob | File processing |
| Queue | Message processing |
| Event Grid | Event-driven |
| Cosmos DB | Change feed processing |

### Best Practices

1. **Keep functions small** and single-purpose
2. **Use Durable Functions** for orchestration
3. **Implement idempotency** for at-least-once triggers
4. **Configure retry policies** appropriately
5. **Monitor with Application Insights**

### Cold Start Mitigation

For Consumption plan:
- Keep package size small (<100MB)
- Use Premium plan for consistent performance
- Implement health check endpoints
- Use pre-warming with timer triggers

## Azure Kubernetes Service (AKS)

### Cluster Configuration

1. **Use managed identity** for cluster authentication
2. **Enable Azure Policy** for governance
3. **Configure network policy** (Calico/Azure)
4. **Use private clusters** for production
5. **Enable container insights** for monitoring

### Node Pool Best Practices

| Pool Type | Use Case |
|-----------|----------|
| System | Core cluster services |
| User (General) | Standard workloads |
| User (Spot) | Batch, interruptible |
| User (GPU) | ML, graphics |

Configure:
- Minimum 3 nodes for production
- Enable cluster autoscaler
- Use availability zones
- Set resource quotas per namespace

### Workload Identity

Replace pod-managed identity with workload identity:
1. Enable OIDC issuer on cluster
2. Create managed identity
3. Establish federated credential
4. Configure service account annotation

### Deployment Patterns

1. **Blue-green**: Two identical environments
2. **Canary**: Gradual rollout with traffic split
3. **Rolling**: Sequential pod replacement
4. **A/B**: Feature-based routing

## Container Apps

### When to Use

- Microservices without Kubernetes complexity
- HTTP APIs and web apps
- Event-driven processing with KEDA
- Background jobs

### Deploying with azd (Recommended)

Container Apps is the primary target for azd deployments:

```bash
# Initialize from Container Apps template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Deploy everything
azd up

# View deployed app
azd show
```

**Popular azd templates for Container Apps:**
- `azure-samples/todo-nodejs-mongo-aca` - Node.js + MongoDB
- `azure-samples/todo-python-mongo-aca` - Python + MongoDB
- `azure-samples/todo-csharp-sql-aca` - .NET + SQL

### Environment Configuration

1. **Use managed certificates** for HTTPS
2. **Configure VNet integration** for private apps
3. **Enable Dapr** for service-to-service communication
4. **Use revision management** for deployment

### Scaling Rules

Configure based on:
- HTTP concurrent requests
- CPU/Memory utilization
- Azure Queue length
- Custom metrics via KEDA

## Common Operations with MCP

### List Web Apps

```
Use azure_appservice_webapp_list to get all web apps.
Use azure_appservice_webapp_get for specific app details.
```

### Manage AKS

```
1. List clusters with azure_aks_cluster_list
2. View node pools with azure_aks_nodepool_list
3. Check cluster status and configuration
```

### Monitor Container Apps

```
Use azure_container_app_list to view container apps.
Check revision status and scaling configuration.
```

## Cost Optimization

1. **Right-size App Service plans** - don't over-provision
2. **Use Consumption Functions** for variable workloads
3. **Enable AKS cluster autoscaler** and scale to zero
4. **Use Spot instances** for non-critical workloads
5. **Reserved instances** for predictable baseline

## Security Checklist

- [ ] Managed identity for all services
- [ ] Key Vault for secrets
- [ ] VNet integration where supported
- [ ] Private endpoints for PaaS services
- [ ] Network security groups
- [ ] Azure Policy for compliance
- [ ] Enable Defender for Cloud

## MCP Tool Reference

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List web apps | `azure_appservice_webapp_list` | Get all web apps |
| Get web app | `azure_appservice_webapp_get` | Get app details |
| List plans | `azure_appservice_plan_list` | Get App Service plans |
| List functions | `azure_function_app_list` | Get function apps |
| List AKS | `azure_aks_cluster_list` | Get AKS clusters |
| List node pools | `azure_aks_nodepool_list` | Get AKS node pools |
| List container apps | `azure_container_app_list` | Get container apps |
