# Azure MCP Plugin

Azure cloud development and operations assistant for Claude Code with progressive skill architecture and integrated MCP server.

**Part of the CHarris Marketplace**

## Features

- **Progressive Skill Architecture**: Single `azure-cloud` skill lazily loads domain-specific content on demand
- **Azure MCP Server Integration**: Direct access to 40+ Azure services via MCP tools
- **Dual Entry Points**: Navigate by domain (data, compute, storage) or by scenario (deployment, diagnostics)
- **Setup Commands**: Easy configuration and authentication helpers
- **Smart Error Handling**: Automatic Docker start, azd subscription selection, CLI tool installation

## Architecture

```
skills/azure-cloud/
├── SKILL.md                    # Root skill (only file with frontmatter)
├── domains/                    # Service-oriented content
│   ├── data/README.md          # Cosmos DB, SQL, Redis
│   ├── compute/README.md       # Container Apps, Functions, AKS
│   ├── storage/README.md       # Blob, Files, Queues
│   ├── security/README.md      # Key Vault, RBAC, Identity
│   ├── ai/README.md            # AI Search, Speech, Foundry
│   ├── networking/README.md    # VNets, Private Endpoints
│   └── observability/README.md # Monitor, App Insights
├── scenarios/                  # Task-oriented content
│   ├── deployment.md           # Deploy applications
│   ├── diagnostics.md          # Debug issues
│   ├── cli-tools.md            # Install az, azd, func
│   ├── cost-optimization.md    # Reduce costs
│   └── security-hardening.md   # Secure resources
├── mcp/                        # MCP server docs
│   ├── setup.md
│   └── tool-reference.md
└── cli/                        # CLI docs
    ├── cheatsheet.md
    └── tips.md
```

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

## Commands

- `/azure:setup` - Configure Azure MCP server and authenticate
- `/azure:auth` - Help with Azure authentication issues
- `/azure:status` - Check MCP server status and available tools

## Usage

The progressive skill architecture means Claude loads only what's needed:

1. **Ask about Azure** - Triggers the `azure-cloud` skill
2. **Skill identifies domain** - Points to relevant domain or scenario file
3. **Claude reads file** - Loads specific guidance on demand
4. **Deep dive if needed** - Service files provide detailed patterns

### Example Interactions

- "Deploy my app to Azure" → Reads `scenarios/deployment.md`
- "Query Cosmos DB" → Reads `domains/data/README.md` → `services/cosmos-db.md`
- "azd not found" → Reads `scenarios/cli-tools.md`
- "Secure my Key Vault" → Reads `domains/security/README.md`

## Authentication

The Azure MCP server uses Azure CLI credentials by default:

```bash
az login
```

For service principal authentication, set environment variables:
- `AZURE_TENANT_ID`
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`

## License

MIT
