# Deploying Applications to Azure

## Choose Your Compute

| If your app is... | Use | Domain File |
|-------------------|-----|-------------|
| HTTP APIs, microservices | Container Apps | `domains/compute/services/container-apps.md` |
| Event-driven functions | Azure Functions | `domains/compute/services/functions.md` |
| Traditional web apps | App Service | `domains/compute/services/app-service.md` |
| Full Kubernetes control | AKS | `domains/compute/services/aks.md` |

## Recommended: Azure Developer CLI (azd)

For new application deployments, use azd:

```bash
# Initialize from template
azd init --template azure-samples/todo-nodejs-mongo-aca

# Provision infrastructure AND deploy code
azd up

# Deploy code only (after initial setup)
azd deploy

# Tear down everything
azd down
```

### Popular Templates

| Template | Stack | Target |
|----------|-------|--------|
| `todo-nodejs-mongo-aca` | Node.js + MongoDB | Container Apps |
| `todo-python-mongo-aca` | Python + MongoDB | Container Apps |
| `todo-csharp-sql-aca` | .NET + SQL | Container Apps |
| `todo-python-mongo-swa-func` | Python + Functions | Static Web Apps |

Browse more: https://azure.github.io/awesome-azd/

## Standard Deployment Flow

1. **Build** - Create container image or package
2. **Push** - Upload to Azure Container Registry
3. **Deploy** - Create/update compute resource
4. **Configure** - Set environment, secrets, scaling
5. **Verify** - Check health, logs, metrics

## Quick Deploy: Container Apps

```bash
# Build and push to ACR
az acr build --registry REGISTRY --image myapp:v1 .

# Deploy
az containerapp up \
  --name myapp \
  --resource-group RG \
  --image REGISTRY.azurecr.io/myapp:v1 \
  --ingress external \
  --target-port 8080
```

## Quick Deploy: App Service

```bash
# ZIP deploy
az webapp deploy \
  --name myapp -g RG \
  --src-path app.zip \
  --type zip
```

## Quick Deploy: Functions

```bash
func azure functionapp publish FUNCTIONAPP
```

## Post-Deployment Checklist

- [ ] Verify application health endpoint
- [ ] Check logs for errors -> `domains/observability/README.md`
- [ ] Set up alerts -> `domains/observability/services/alerts.md`
- [ ] Configure custom domain if needed
- [ ] Review security settings -> `scenarios/security-hardening.md`
- [ ] Set up CI/CD pipeline

## Troubleshooting Deployments

**Application not starting:**
1. Check container logs: `az containerapp logs show`
2. Verify environment variables
3. Check for startup errors in App Insights

**Health check failing:**
1. Verify health endpoint returns 200
2. Check if port is correct
3. Review application startup time

**Permission errors:**
1. Check managed identity is enabled
2. Verify RBAC role assignments
3. Review Key Vault access policies
