# Installing Azure CLI Tools

## Overview

Azure development requires specific CLI tools. This guide helps detect missing tools and provides installation guidance.

## Primary Tools

| Tool | Name | Purpose |
|------|------|---------|
| `az` | Azure CLI | Resource management, queries, scripting |
| `azd` | Azure Developer CLI | Application deployment (infrastructure + code) |
| `func` | Azure Functions Core Tools | Local Functions development |

## Using the MCP Install Tool

**When a CLI tool is missing, use the Azure MCP server's installation tool:**

```
Tool: azure__extension_cli_install
Parameters:
  - cli-type: "az" | "azd" | "func"
```

### Get Installation Instructions

For Azure CLI (az):
```
Use azure__extension_cli_install with cli-type: "az"
```

For Azure Developer CLI (azd):
```
Use azure__extension_cli_install with cli-type: "azd"
```

For Functions Core Tools (func):
```
Use azure__extension_cli_install with cli-type: "func"
```

## Quick Installation Commands

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

## Verification

After installation, verify the tools are working:

```bash
# Azure CLI
az version

# Azure Developer CLI
azd version

# Azure Functions Core Tools
func --version
```

## Authentication

### Azure CLI (az)

```bash
# Interactive login (opens browser)
az login

# Device code flow (for remote/headless)
az login --use-device-code

# Service principal
az login --service-principal -u APP_ID -p PASSWORD --tenant TENANT_ID
```

### Azure Developer CLI (azd)

```bash
# Uses Azure CLI credentials by default
# Or explicitly login
azd auth login
```

## Error Patterns to Detect

**Azure CLI (az) not installed:**
- `'az' is not recognized as an internal or external command`
- `az: command not found`

**Azure Developer CLI (azd) not installed:**
- `'azd' is not recognized as an internal or external command`
- `azd: command not found`

**Azure Functions Core Tools (func) not installed:**
- `'func' is not recognized as an internal or external command`
- `func: command not found`

## When to Install Which Tool

| Task | Required Tool |
|------|---------------|
| List resources, manage subscriptions | `az` |
| Deploy applications to Azure | `azd` |
| Develop Azure Functions locally | `func` |
| Query Azure resources | `az` |
| Infrastructure as Code deployment | `az` or `azd` |

## Common Issues

### Path Issues After Installation

**Windows:** Restart terminal or run `refreshenv`

**macOS/Linux:** Run `source ~/.bashrc` or `source ~/.zshrc`

### Permission Issues

**Windows:** Run terminal as Administrator

**macOS/Linux:** Installation may require `sudo`
