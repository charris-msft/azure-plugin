# Azure MCP Plugin Development Guidelines

## ⚠️ CRITICAL: Docker Daemon Errors - Auto-Start Required

**When ANY Docker error occurs (daemon not running, cannot connect, etc.), you MUST:**

1. **DO NOT** just tell the user to start Docker manually
2. **DO** ask: "Docker isn't running. Would you like me to start Docker Desktop for you?"
3. **If yes**, run the appropriate command:
   - **Windows:** `"/c/Program Files/Docker/Docker/Docker Desktop.exe" &`
   - **macOS:** `open -a Docker`
   - **Linux:** `sudo systemctl start docker`
4. **Wait** for Docker to be ready (30-60 seconds):
   ```bash
   for i in {1..30}; do docker info >/dev/null 2>&1 && echo "Docker is ready!" && break; sleep 2; done
   ```
5. **Retry** the original command automatically

**Error patterns to detect:**
- `docker daemon is not running`
- `Cannot connect to the Docker daemon`
- `failed to connect to the docker API`
- `The system cannot find the file specified` (Docker pipe error)
- `Is the docker daemon running?`

---

## azd Commands - ALWAYS Use Non-Interactive Mode

**CRITICAL: azd commands MUST be run non-interactively. Claude cannot provide interactive input.**

**Before ANY azd command:**
1. **Create environment first:** `azd env new <env-name> --no-prompt`
2. **Set required values:**
   ```bash
   azd env set AZURE_SUBSCRIPTION_ID "<subscription-id>"
   azd env set AZURE_LOCATION "<location>"
   ```
3. **Always use `--no-prompt`:** All azd commands should include `--no-prompt`

**Correct patterns:**
```bash
# Initialize with environment name
azd init --environment my-app-dev --no-prompt

# Or create environment separately first
azd env new my-app-dev --no-prompt
azd env set AZURE_SUBSCRIPTION_ID "xxxx-xxxx-xxxx"
azd env set AZURE_LOCATION "eastus2"

# Then provision/deploy
azd up --no-prompt
azd provision --no-prompt
azd deploy --no-prompt
```

**NEVER run these without --no-prompt or pre-configured environment:**
- `azd init` (prompts for environment name)
- `azd up` (prompts for environment, subscription, location)
- `azd provision` (prompts for subscription)

**Error pattern to watch for:**
- `environment name '' is invalid` = infinite loop, need to specify environment
- `Enter a unique environment name` = stuck waiting for input

---

## azd Subscription Selection - Handle Gracefully

**Before running `azd provision` or `azd up`, ALWAYS check if subscription is configured:**

```bash
# Check if AZURE_SUBSCRIPTION_ID is set in the azd environment
cat .azure/<env-name>/.env 2>/dev/null | grep AZURE_SUBSCRIPTION_ID
```

**If no subscription is set:**
1. **DO NOT** just run `azd provision` and let it fail with the giant subscription list
2. **DO** use `az account list --output table` to get a clean list
3. **DO** ask the user which subscription to use with AskUserQuestion
4. **DO** set it with: `azd env set AZURE_SUBSCRIPTION_ID <subscription-id>`
5. **Then** run `azd provision`

**Quick subscription setup:**
```bash
# List subscriptions cleanly
az account list --query "[].{Name:name, ID:id}" -o table

# Set subscription in azd environment
azd env set AZURE_SUBSCRIPTION_ID "<subscription-id>"
```

---

## Version Management

**IMPORTANT: Always increment the plugin version when making changes.**

After ANY modification to plugin files (skills, commands, hooks, etc.):

1. Update `.claude-plugin/plugin.json` version field
2. Use semantic versioning:
   - **PATCH** (0.1.x): Bug fixes, typo corrections, minor updates
   - **MINOR** (0.x.0): New features, new skills/commands, significant improvements
   - **MAJOR** (x.0.0): Breaking changes, major restructuring

## Plugin Structure

```
azure-mcp/
├── .claude-plugin/plugin.json   # Manifest (bump version here!)
├── .mcp.json                    # Azure MCP server config
├── CLAUDE.md                    # This file
├── README.md                    # User documentation
├── commands/                    # Slash commands
├── hooks/                       # Event hooks
└── skills/                      # Best practices skills
```

## Development Practices

### Adding/Modifying Skills

- Use third-person descriptions ("This skill should be used when...")
- Keep SKILL.md lean (1,500-2,000 words)
- Use imperative form in body text
- Include specific trigger phrases in description

### Deployment Guidance

- Prefer `azd` (Azure Developer CLI) for application deployments
- Use `az` for resource management and queries
- Document both options where applicable

### Windows Compatibility

- Test commands work in Git Bash (MINGW64)
- Check for PATH issues with Azure CLI
- Use Node.js for cross-platform hook scripts (not bash)

## Changelog

### 1.3.0
- **NEW: Pre-deployment validation scenario** (`scenarios/validation.md`)
- **Azure naming constraints** - Documents 24-char limits for Storage/KeyVault, character restrictions
- **PostToolUse hook catches naming errors** - Detects "must be between 3 and 24 characters", "already in use", etc.
- **PreToolUse hook warns on large output** - Blocks `az account list` without `--query` to prevent 140K char overflow
- **Bicep validation guidance** - Use `azure__bicepschema_get` and `azure__deploy_iac_rules_get` before writing IaC
- **Deployment plan generation** - Use `azure__deploy_plan_get` for comprehensive deployment planning
- **Resource filtering best practices** - Always use `--query` and `-o table` for az commands

### 1.2.0
- **CRITICAL: PreToolUse hook enforces azd over az for deployments**
- Hook blocks `az containerapp create`, `az webapp up`, `az functionapp create` etc.
- Hook allows `az` for queries only (show, list, logs)
- Updated skill description to make azd mandatory
- Strengthened all deployment documentation with "MANDATORY" language
- `az` should only be used for deployments when user explicitly requests it

### 1.1.0
- **NEW: /azure:preflight command** for pre-deployment checks (tools, auth, quotas)
- **NEW: Node.js/Express production scenario** with trust proxy, cookie config, health checks
- **IMPROVED: azd strongly preferred** over az for all deployments (parallel provisioning is faster)
- **IMPROVED: ACR integration guidance** - automatic credential config, image pull troubleshooting
- **IMPROVED: ACR Tasks fallback** - detect disabled ACR Tasks (free subs), offer local Docker build
- **IMPROVED: CLI auto-installation** - detect missing tools, offer one-click winget/brew install
- **IMPROVED: Container Apps troubleshooting** - cold starts, port mismatch, health probe failures
- **IMPROVED: Hooks** detect ACR Tasks errors, CLI not found, image pull failures
- **ADDED: azd down --force --purge** guidance for test environment cleanup

### 1.0.0
- **MAJOR: Progressive Skill Architecture**
- Consolidated 7 individual skills into single `azure-cloud` skill
- Only root SKILL.md has YAML frontmatter (~300 tokens in context)
- Domain content loads lazily on demand (data, compute, storage, security, ai, networking, observability)
- Dual entry points: by domain (service taxonomy) or by scenario (task-based)
- Added scenarios: deployment, diagnostics, cli-tools, cost-optimization, security-hardening
- Added mcp/ directory with setup and tool-reference docs
- Added cli/ directory with cheatsheet and tips
- Service-specific files under each domain (e.g., cosmos-db.md, container-apps.md)

### 0.6.0
- **NEW: Azure CLI Tools skill** for detecting and installing missing CLI tools
- Skill triggers on "az not found", "azd not found", "func not found" errors
- Guides Claude to use `azure__extension_cli_install` MCP tool for installation
- Covers Azure CLI (az), Azure Developer CLI (azd), and Functions Core Tools (func)
- Platform-specific installation commands as fallback

### 0.5.0
- **CRITICAL FIX: azd non-interactive mode enforcement**
- Added guidance to ALWAYS use `--no-prompt` with azd commands
- Added hook to detect azd interactive input loops (`environment name '' is invalid`)
- Hook now catches 3 error types: Docker, azd subscription, azd interactive input
- Proper workflow: `azd env new` → `azd env set` → `azd up --no-prompt`

### 0.4.1
- Added azd subscription error handling to PostToolUse hook
- Catches "is not an allowed choice" errors from azd and provides graceful recovery
- Added CLAUDE.md guidance for checking subscription before azd provision/up

### 0.4.0
- **NEW: PostToolUse hook** to automatically detect Docker errors in Bash output
- Hook intercepts Docker daemon errors and instructs Claude to use AskUserQuestion
- This ENFORCES Docker auto-start behavior rather than relying on soft guidance
- CLAUDE.md guidance retained as backup

### 0.3.3
- Made Docker auto-start guidance CRITICAL and first section in CLAUDE.md
- Added explicit "DO NOT tell user to start Docker manually" instruction
- Added Docker pipe error pattern detection
- Consolidated duplicate Docker sections

### 0.3.2
- Added Docker auto-start guidance to CLAUDE.md so Claude always offers to start Docker when encountering daemon errors

### 0.3.1
- Auto-start Docker Desktop when not running (with user consent)
- Wait loop for Docker to become ready
- Platform-specific start commands (Windows, macOS, Linux)

### 0.3.0
- Added Docker detection and setup guidance to setup command
- Handle Docker not installed, not running, and permission issues
- Platform-specific Docker installation instructions
- Docker troubleshooting section

### 0.2.0
- Added azd (Azure Developer CLI) as preferred deployment tool
- Updated azure-compute skill with azd guidance
- Updated azure-infrastructure skill with azd vs az comparison
- Enhanced setup command for Windows/Git Bash compatibility

### 0.1.0
- Initial release
- 6 skills: storage, compute, data, security, ai, infrastructure
- 3 commands: setup, auth, status
- Azure MCP server integration
- Auto-enable reminder hook
