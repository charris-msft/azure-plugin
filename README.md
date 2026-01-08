# Azure MCP Plugin

Azure development assistant plugin for Claude Code with integrated MCP server and comprehensive best practices skills.

**Part of the CHarris Marketplace**

## Features

- **Azure MCP Server Integration**: Direct access to 40+ Azure services via MCP tools
- **Best Practices Skills**: Comprehensive guidance for major Azure service categories
- **Setup Commands**: Easy configuration and authentication helpers
- **Auto-Enable Prompts**: Intelligent reminders when Azure MCP tools are needed
- **Smart Error Handling**: Automatic Docker start, azd subscription selection, non-interactive mode enforcement

## Prerequisites

- Node.js 18+ (for npx)
- Azure CLI installed and configured
- Azure subscription with appropriate permissions
- Docker Desktop (optional, for container deployments)

## Installation

### Option 1: Via Marketplace (Recommended)

```bash
# Add the CHarris Marketplace
/plugin marketplace add charris-msft/azure-plugin

# Install the Azure MCP plugin
/plugin install azure-mcp@charris-marketplace
```

### Option 2: Direct from GitHub

```bash
/plugin install charris-msft/azure-plugin
```

### Option 3: Manual Installation

1. Clone this repository
2. Run Claude Code with `--plugin-dir ./azure-mcp`
3. Enable the plugin in Claude Code settings

### After Installation

Run `/azure:setup` to configure authentication and verify everything is working.

## Skills Included

| Skill | Description |
|-------|-------------|
| `azure-storage` | Blob storage, containers, queues, file shares |
| `azure-compute` | App Service, Functions, AKS, Container Apps |
| `azure-data` | SQL Database, Cosmos DB, Redis Cache |
| `azure-security` | Key Vault, RBAC, managed identities |
| `azure-ai` | AI Search, Cognitive Services, Foundry |
| `azure-infrastructure` | CLI, Resource Groups, ARM/Bicep deployments |

## Commands

- `/azure:setup` - Configure Azure MCP server and authenticate
- `/azure:auth` - Help with Azure authentication issues
- `/azure:status` - Check MCP server status and available tools

## Authentication

The Azure MCP server uses Azure CLI credentials by default. Run:

```bash
az login
```

For service principal authentication, set environment variables:
- `AZURE_TENANT_ID`
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`

## Usage

Once enabled, simply ask Claude about Azure services:

- "List my storage accounts"
- "What containers are in my storage account?"
- "Show me my AKS clusters"
- "Create a new resource group"

The plugin will automatically activate relevant skills and suggest enabling the MCP server when needed.

## License

MIT
