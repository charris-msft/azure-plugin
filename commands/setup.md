---
name: setup
description: Configure Azure MCP server and authenticate to Azure
argument-hint: "[--check-only]"
allowed-tools: ["Bash", "Read", "Write", "AskUserQuestion", "mcp__azure__*"]
---

# Azure Setup Command

Guide the user through setting up and configuring the Azure MCP server for Claude Code.

## Steps

### 1. Detect Shell Environment

First, detect what shell environment is being used:

```bash
uname -a
```

Interpret the output:
- **MINGW64** or **MSYS** = Git Bash on Windows
- **Linux** with "microsoft" or "WSL" in kernel = WSL on Windows
- **Darwin** = macOS
- **Linux** (other) = Native Linux

Store this context for providing appropriate instructions throughout setup.

### 2. Check Node.js

```bash
node --version
```

If Node.js is not installed:
- Inform user Node.js is required for Azure MCP server
- Download from https://nodejs.org/ (recommend LTS version)
- Wait for user to install before proceeding

### 3. Check Azure CLI Availability

This is the critical step that requires platform-specific handling.

#### Step 3a: Check if `az` is in PATH

```bash
az version 2>&1
```

If this succeeds, proceed to Step 4.

#### Step 3b: If `az` not found on Git Bash (MINGW64)

When running on Git Bash but `az` command fails, check if Azure CLI is installed on Windows:

```bash
# Check common Windows installation paths
ls "/c/Program Files/Microsoft SDKs/Azure/CLI2/wbin/az.cmd" 2>/dev/null
```

**If found (installed on Windows but not in Git Bash PATH):**

Present the user with options using AskUserQuestion:

**Option 1: Add to PATH (Recommended)**
Explain this adds Azure CLI to Git Bash permanently:
```bash
echo 'export PATH="$PATH:/c/Program Files/Microsoft SDKs/Azure/CLI2/wbin"' >> ~/.bashrc
source ~/.bashrc
```

**Option 2: Use full path**
Can invoke az using the full path:
```bash
"/c/Program Files/Microsoft SDKs/Azure/CLI2/wbin/az.cmd" version
```

After user chooses and applies fix, verify `az version` works.

#### Step 3c: If Azure CLI not installed anywhere

Check for Azure Developer CLI (azd) as alternative:
```bash
azd version 2>&1
```

**Present options to user using AskUserQuestion:**

```
Azure CLI (az) is not installed. Choose how to proceed:

Option 1: Install Azure CLI (Recommended)
- Provides full Azure management capabilities
- Required for Azure MCP server
- Installation requires admin approval (UAC prompt on Windows)

Option 2: Skip for now
- Can still use Azure skills for guidance
- MCP tools won't be available until az is installed
```

**If user chooses to install:**

For **Git Bash on Windows**:
```
To install Azure CLI on Windows:

1. Open PowerShell or Command Prompt as Administrator
2. Run: winget install Microsoft.AzureCLI
3. Accept the UAC prompt when it appears
4. Close and reopen your terminal after installation

I cannot run this installation from Git Bash - it must be done from Windows directly.
Would you like me to wait while you install it?
```

For **WSL**:
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

For **macOS**:
```bash
brew install azure-cli
```

For **Linux**:
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### 4. Check Docker (Optional but Recommended)

Docker is needed for building and testing containers (Container Apps, AKS, Functions with containers).

#### Step 4a: Check if Docker is installed

```bash
docker --version 2>&1
```

#### Step 4b: If Docker not found

**Present options to user using AskUserQuestion:**

```
Docker is not installed. Docker is optional but recommended for:
- Building container images
- Testing containers locally
- Deploying to Container Apps or AKS

Would you like to install Docker?
```

**If user chooses to install:**

For **Windows (Git Bash or PowerShell):**
```
To install Docker Desktop on Windows:

1. Download from https://www.docker.com/products/docker-desktop/
2. Run the installer
3. Restart your computer after installation
4. Start Docker Desktop from the Start menu

I cannot install Docker automatically - it requires manual download and admin rights.
```

For **WSL:**
```
Docker Desktop for Windows with WSL 2 backend is recommended:

1. Install Docker Desktop on Windows (not inside WSL)
2. Enable WSL 2 integration in Docker Desktop settings
3. Docker commands will then work inside WSL

Alternatively, install Docker Engine directly in WSL:
sudo apt-get update
sudo apt-get install docker.io
sudo usermod -aG docker $USER
```

For **macOS:**
```bash
# Using Homebrew
brew install --cask docker

# Then start Docker Desktop from Applications
```

For **Linux:**
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

#### Step 4c: If Docker installed but not running

```bash
docker info 2>&1
```

If error contains "Cannot connect to the Docker daemon" or "docker daemon is not running":

**On Windows - Check if Docker Desktop is installed:**
```bash
ls "/c/Program Files/Docker/Docker/Docker Desktop.exe" 2>/dev/null
```

**If Docker Desktop is installed, ask user using AskUserQuestion:**
```
Docker Desktop is installed but not running.

Would you like me to start Docker Desktop for you?
```

**If user agrees, start Docker Desktop:**

For **Git Bash on Windows:**
```bash
"/c/Program Files/Docker/Docker/Docker Desktop.exe" &
```

For **PowerShell/CMD:**
```powershell
Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
```

**Then wait for Docker to be ready** (can take 30-60 seconds):
```bash
echo "Starting Docker Desktop... this may take up to 60 seconds"
for i in {1..30}; do
  if docker info >/dev/null 2>&1; then
    echo "Docker is ready!"
    break
  fi
  sleep 2
done
```

If Docker still not ready after waiting, inform user:
```
Docker Desktop is starting but not ready yet.
Please wait for the whale icon in the system tray to stop animating.
```

**On macOS:**
```bash
open -a Docker
# Then wait for it to be ready
```

**On Linux:**
```bash
sudo systemctl start docker
```

After Docker is running, verify with `docker info`.

#### Step 4d: Docker running successfully

If Docker is running, optionally verify it works:
```bash
docker run --rm hello-world
```

**Note:** Docker is optional. If user skips Docker setup, continue with the rest of setup but note in summary that container features won't be available.

### 5. Check Azure Authentication

```bash
az account show 2>&1
```

If not authenticated (error contains "Please run 'az login'"):

```bash
az login
```

If browser doesn't open or user is in a headless environment:
```bash
az login --use-device-code
```

Guide user through the device code flow:
1. Open https://microsoft.com/devicelogin
2. Enter the code displayed
3. Complete sign-in in browser

For service principal authentication (CI/CD scenarios):
```bash
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>
```

### 6. Verify Subscription

```bash
az account list --output table
```

If multiple subscriptions exist, ask user which one to use:

```bash
az account set --subscription "<subscription-name-or-id>"
```

Confirm the selection:
```bash
az account show --query "{Name:name, ID:id}" -o table
```

### 7. Enable Azure MCP Server

Check if Azure MCP tools are available by attempting to use one.

If MCP tools are not available:
1. Inform user: "The Azure MCP server needs to be enabled"
2. Direct them to run `/mcp`
3. Look for "azure" in the server list
4. Enable it
5. Return here to continue

### 8. Test Connection

Once MCP is enabled, verify the connection:

Use `azure_subscription_list` MCP tool to list subscriptions.

If successful, the MCP server is working correctly.

### 9. Display Summary

After setup completes, display a summary:

```
Azure Setup Complete!
=====================

Environment
  Shell: Git Bash (MINGW64)
  Node.js: v20.x.x ✓

Azure CLI
  Version: 2.x.x ✓
  Status: Authenticated ✓

Docker
  Version: 24.x.x ✓       (or "Not installed - container features unavailable")
  Status: Running ✓       (or "Not running" or "Skipped")

Subscription
  Name: Contoso Production
  ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

Azure MCP Server
  Status: Enabled ✓
  Tools: 40+ available

Ready to use Azure tools!

Try asking:
  • "List my storage accounts"
  • "Show my resource groups"
  • "What AKS clusters do I have?"
```

## Troubleshooting

### Azure CLI not found in Git Bash

**Symptom:** `az: command not found` in Git Bash, but works in PowerShell

**Cause:** Azure CLI installed on Windows isn't in Git Bash's PATH

**Solutions:**

1. **Add to PATH permanently:**
   ```bash
   echo 'export PATH="$PATH:/c/Program Files/Microsoft SDKs/Azure/CLI2/wbin"' >> ~/.bashrc
   source ~/.bashrc
   ```

2. **Create an alias:**
   ```bash
   echo 'alias az="/c/Program Files/Microsoft SDKs/Azure/CLI2/wbin/az.cmd"' >> ~/.bashrc
   source ~/.bashrc
   ```

### Authentication Issues

If `az login` fails:
1. Check network connectivity
2. Try `az login --use-device-code` for browser issues
3. Verify Azure AD tenant is accessible
4. Check if MFA is required (use device code flow)

### MCP Server Not Starting

If Azure MCP server fails:
1. Verify Node.js is installed: `node --version`
2. Verify npx is available: `npx --version`
3. Check for firewall/proxy blocking npm registry
4. Try running manually: `npx -y @azure/mcp@latest server start`

### Permission Errors (403)

If operations fail with 403:
1. Verify RBAC role assignments in Azure Portal
2. Check subscription-level permissions
3. Confirm correct subscription is selected
4. Some operations require Owner or specific roles

### WSL vs Git Bash Confusion

**Symptom:** Different az installations in different shells

**Explanation:** WSL and Git Bash are separate environments. Azure CLI installed in one won't be available in the other.

**Solution:** Install Azure CLI in both environments if you use both, or stick to one shell for Azure work.

### Docker Issues

#### Docker not found

**Windows:** Download and install Docker Desktop from https://www.docker.com/products/docker-desktop/

**macOS:** `brew install --cask docker` or download Docker Desktop

**Linux:** `curl -fsSL https://get.docker.com | sudo sh`

#### Docker daemon not running

**Symptom:** `Cannot connect to the Docker daemon` or `docker daemon is not running`

**Solutions:**

- **Windows (auto-start):**
  ```bash
  # From Git Bash
  "/c/Program Files/Docker/Docker/Docker Desktop.exe" &

  # From PowerShell
  Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
  ```

- **macOS (auto-start):**
  ```bash
  open -a Docker
  ```

- **Linux:**
  ```bash
  sudo systemctl start docker
  ```

**Wait for Docker to be ready** (30-60 seconds on Windows/macOS):
```bash
for i in {1..30}; do docker info >/dev/null 2>&1 && echo "Ready!" && break; sleep 2; done
```

#### Permission denied on Linux

**Symptom:** `permission denied while trying to connect to the Docker daemon socket`

**Solution:**
```bash
sudo usermod -aG docker $USER
# Log out and back in, or run:
newgrp docker
```

#### Docker Desktop not starting on Windows

1. Ensure WSL 2 is installed: `wsl --install`
2. Ensure virtualization is enabled in BIOS
3. Try restarting Docker Desktop
4. Check Windows Event Viewer for errors

#### Docker in WSL without Docker Desktop

If you don't want Docker Desktop, you can run Docker Engine directly in WSL:
```bash
sudo apt-get update
sudo apt-get install docker.io docker-compose
sudo service docker start
sudo usermod -aG docker $USER
```
