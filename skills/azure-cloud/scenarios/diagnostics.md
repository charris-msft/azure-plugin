# Debugging Production Issues

## Quick Diagnosis Flow

1. **Identify symptoms** - What's failing?
2. **Check resource health** - Is Azure healthy?
3. **Review logs** - What do logs show?
4. **Analyze metrics** - Performance patterns?
5. **Investigate recent changes** - What changed?

## Check Azure Resource Health

### Using MCP

```
Use azure_resourcehealth_* tools to check resource availability status.
```

### Using CLI

```bash
# Check specific resource health
az resource show --ids RESOURCE_ID

# Check recent activity
az monitor activity-log list -g RG --max-events 20
```

## Log Analysis

### Application Logs

**Container Apps:**
```bash
az containerapp logs show --name APP -g RG --follow
```

**App Service:**
```bash
az webapp log tail --name APP -g RG
```

**Functions:**
```bash
func azure functionapp logstream FUNCTIONAPP
```

### Log Analytics (KQL)

Common diagnostic queries:

```kql
// Recent errors
AppExceptions
| where TimeGenerated > ago(1h)
| project TimeGenerated, Message, StackTrace
| order by TimeGenerated desc

// Failed requests
AppRequests
| where Success == false
| where TimeGenerated > ago(1h)
| summarize count() by Name, ResultCode
| order by count_ desc

// Slow requests
AppRequests
| where TimeGenerated > ago(1h)
| where DurationMs > 5000
| project TimeGenerated, Name, DurationMs
| order by DurationMs desc

// Dependency failures
AppDependencies
| where Success == false
| where TimeGenerated > ago(1h)
| summarize count() by Name, ResultCode, Target
```

## Common Issues by Service

### Container Apps

| Symptom | Check |
|---------|-------|
| 502/503 errors | Container logs, health probe |
| Slow response | Scaling rules, resource limits |
| Not starting | Environment variables, image pull |

### App Service

| Symptom | Check |
|---------|-------|
| 503 Service Unavailable | App logs, memory/CPU usage |
| Slow cold start | Always On setting, app startup |
| Deployment failures | Deployment logs, slot swap |

### Azure Functions

| Symptom | Check |
|---------|-------|
| Not triggering | Trigger configuration, host.json |
| Timeout errors | Execution time, plan limits |
| Cold starts | Premium plan, package size |

### Database Issues

| Symptom | Check |
|---------|-------|
| Connection failures | Firewall rules, connection limits |
| Slow queries | Query Performance Insights |
| Throttling | DTU/RU usage, tier limits |

## Metrics to Monitor

### Application Metrics

```bash
# Get App Service metrics
az monitor metrics list \
  --resource RESOURCE_ID \
  --metric "Http5xx" "AverageResponseTime" "CpuPercentage"
```

### Key Metrics by Service

| Service | Critical Metrics |
|---------|-----------------|
| Container Apps | Requests, Replicas, CPU, Memory |
| App Service | HTTP 5xx, Response Time, CPU |
| Functions | Executions, Failures, Duration |
| SQL Database | DTU %, Connections, Deadlocks |
| Cosmos DB | RU consumption, 429 errors |

## Using AppLens (MCP)

For comprehensive diagnostics:

```
Use azure__applens tools to get AI-powered diagnostics and recommendations.
```

AppLens provides:
- Automated issue detection
- Root cause analysis
- Remediation recommendations

## Escalation Checklist

Before escalating:
- [ ] Checked resource health status
- [ ] Reviewed application logs
- [ ] Analyzed recent deployments
- [ ] Checked for Azure service issues
- [ ] Reviewed metric dashboards
- [ ] Attempted basic remediation
