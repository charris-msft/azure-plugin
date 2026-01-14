# Azure Container Apps

## Quick Reference

| Property | Value |
|----------|-------|
| CLI prefix | `az containerapp` |
| MCP tools | `azure_container_app_list` |
| Best for | Microservices, serverless containers, HTTP APIs |

## When to Use Container Apps

- Microservices without Kubernetes complexity
- HTTP APIs and web apps
- Event-driven processing with KEDA
- Background jobs and workers

## Deploying with azd (Recommended)

```bash
# Initialize from template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Deploy everything
azd up

# View deployed app
azd show
```

**Popular azd templates:**
- `azure-samples/todo-nodejs-mongo-aca` - Node.js + MongoDB
- `azure-samples/todo-python-mongo-aca` - Python + MongoDB
- `azure-samples/todo-csharp-sql-aca` - .NET + SQL

## CLI Deployment

```bash
# Build and push to ACR
az acr build --registry REGISTRY --image myapp:v1 .

# Deploy container app
az containerapp up \
  --name myapp \
  --resource-group RG \
  --image REGISTRY.azurecr.io/myapp:v1 \
  --ingress external \
  --target-port 8080

# Update with new image
az containerapp update \
  --name myapp \
  --resource-group RG \
  --image REGISTRY.azurecr.io/myapp:v2
```

## Scaling Configuration

### HTTP Scaling

```bash
az containerapp update \
  --name myapp \
  --resource-group RG \
  --min-replicas 1 \
  --max-replicas 10 \
  --scale-rule-name http-rule \
  --scale-rule-type http \
  --scale-rule-http-concurrency 50
```

### KEDA Scaling

Supports Azure Queue, Service Bus, Cosmos DB change feed, and more.

## Environment Variables and Secrets

```bash
# Set environment variable
az containerapp update \
  --name myapp -g RG \
  --set-env-vars KEY=VALUE

# Create secret
az containerapp secret set \
  --name myapp -g RG \
  --secrets dbpassword=secretvalue

# Reference secret in env var
az containerapp update \
  --name myapp -g RG \
  --set-env-vars DB_PASSWORD=secretref:dbpassword
```

## Dapr Integration

Enable Dapr for service-to-service communication:

```bash
az containerapp update \
  --name myapp -g RG \
  --enable-dapr \
  --dapr-app-id myapp \
  --dapr-app-port 8080
```

## Revision Management

```bash
# List revisions
az containerapp revision list --name myapp -g RG --output table

# Split traffic
az containerapp ingress traffic set \
  --name myapp -g RG \
  --revision-weight myapp--rev1=80 myapp--rev2=20
```

## Best Practices

1. Use managed certificates for HTTPS
2. Configure VNet integration for private apps
3. Enable Dapr for service mesh features
4. Use revision-based deployments for rollback
5. Store secrets in Key Vault with managed identity
