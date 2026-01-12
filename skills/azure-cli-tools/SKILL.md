---
name: Azure CLI Tools
description: This skill should be used when the user asks to "install Azure CLI", "install azd", "install func", "Azure CLI not found", "azd not found", "func not found", "command not found az", "command not found azd", "command not found func", "setup Azure development tools", "Azure developer tools", or when any Azure CLI tool (az, azd, func) is missing or needs installation. Also use when seeing errors like "'az' is not recognized", "'azd' is not recognized", or "func: command not found". Provides installation guidance using the Azure MCP server's CLI install tool.
---

# Azure CLI Tools Installation

## Overview

Azure development requires specific CLI tools. This skill helps detect missing tools and provides installation guidance using the Azure MCP server.

**Primary Tools:**

| Tool | Name | Purpose |
|------|------|---------|
| `az` | Azure CLI | Resource management, queries, scripting |
| `azd` | Azure Developer CLI | Application deployment (infrastructure + code) |
| `func` | Azure Functions Core Tools | Local Functions development and deployment |

## Detecting Missing CLI Tools

### Common Error Patterns

**Azure CLI (az) not installed:**
- `'az' is not recognized as an internal or external command`
- `az: command not found`
- `The term 'az' is not recognized`

**Azure Developer CLI (azd) not installed:**
- `'azd' is not recognized as an internal or external command`
- `azd: command not found`
- `The term 'azd' is not recognized`

**Azure Functions Core Tools (func) not installed:**
- `'func' is not recognized as an internal or external command`
- `func: command not found`
- `The term 'func' is not recognized`

## Using the MCP Install Tool

**When a CLI tool is missing, use the Azure MCP server's installation tool:**

```
Tool: azure__extension_cli_install
Parameters:
  - cli-type: "az" | "azd" | "func"
```

### Getting Installation Instructions

**For Azure CLI (az):**
```
Use azure__extension_cli_install with cli-type: "az"
```

**For Azure Developer CLI (azd):**
```
Use azure__extension_cli_install with cli-type: "azd"
```

**For Azure Functions Core Tools (func):**
```
Use azure__extension_cli_install with cli-type: "func"
```

## Quick Installation Commands

If the MCP tool is not available, use these platform-specific commands:

### Azure CLI (az)

**Windows:**
```powershell
winget install Microsoft.AzureCLI
```

**macOS:**
```bash
brew install azure-cli
```

**Linux (Ubuntu/Debian):**
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### Azure Developer CLI (azd)

**Windows:**
```powershell
winget install Microsoft.Azd
```

**macOS:**
```bash
brew tap azure/azd && brew install azd
```

**Linux:**
```bash
curl -fsSL https://aka.ms/install-azd.sh | bash
```

### Azure Functions Core Tools (func)

**Windows:**
```powershell
winget install Microsoft.Azure.FunctionsCoreTools
```

**macOS:**
```bash
brew tap azure/functions && brew install azure-functions-core-tools@4
```

**Linux (Ubuntu/Debian):**
```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update
sudo apt-get install azure-functions-core-tools-4
```

## Verification Commands

After installation, verify the tools are working:

```bash
# Azure CLI
az version

# Azure Developer CLI
azd version

# Azure Functions Core Tools
func --version
```

## Authentication After Installation

### Azure CLI (az)

```bash
# Interactive login (opens browser)
az login

# Device code flow (for remote/headless)
az login --use-device-code

# Service principal
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>
```

### Azure Developer CLI (azd)

```bash
# Uses Azure CLI credentials by default
# Or explicitly login
azd auth login
```

### Azure Functions Core Tools (func)

```bash
# Functions Core Tools uses Azure CLI credentials
# No separate login required
```

## When to Install Which Tool

| Task | Required Tool |
|------|---------------|
| List resources, manage subscriptions | `az` |
| Deploy applications to Azure | `azd` |
| Develop Azure Functions locally | `func` |
| Query Azure resources | `az` |
| Infrastructure as Code deployment | `az` or `azd` |
| Create Function App from template | `func` |
| Container Apps deployment | `azd` |

## Workflow: Handling Missing Tools

1. **Detect the error** - Recognize CLI not found patterns
2. **Identify the tool** - Determine which CLI is needed (az, azd, or func)
3. **Get instructions** - Use `azure__extension_cli_install` MCP tool
4. **Provide to user** - Share platform-specific installation steps
5. **Verify installation** - Run version check command
6. **Authenticate** - Help user login if needed

## Generating CLI Commands

The Azure MCP server also provides a tool to generate CLI commands:

```
Tool: azure__extension_cli_generate
Parameters:
  - intent: Description of what you want to do
  - cli-type: "az"
```

Use this when you need to construct complex Azure CLI commands.

## Common Issues

### Path Issues After Installation

**Windows:** Restart terminal or run `refreshenv`

**macOS/Linux:** Run `source ~/.bashrc` or `source ~/.zshrc`

### Permission Issues

**Windows:** Run terminal as Administrator for installation

**macOS/Linux:** Installation scripts may require `sudo`

### Multiple Versions

**Azure Functions Core Tools** has multiple versions (v3, v4). Use v4 for new projects:
- Windows: `azure-functions-core-tools-4`
- macOS: `azure-functions-core-tools@4`

## MCP Tool Reference

| Operation | MCP Tool | cli-type |
|-----------|----------|----------|
| Install Azure CLI | `azure__extension_cli_install` | `az` |
| Install Azure Developer CLI | `azure__extension_cli_install` | `azd` |
| Install Functions Core Tools | `azure__extension_cli_install` | `func` |
| Generate CLI commands | `azure__extension_cli_generate` | `az` |
