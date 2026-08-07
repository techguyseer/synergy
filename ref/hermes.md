# Hermes Agent Installation & Integration Guide

This guide provides comprehensive, step-by-step instructions for installing, configuring, and deploying the **Hermes Agent** as the autonomous execution layer of the **Synergy Marketing Ecosystem** — running on a separate environment (local machine, WSL2, dedicated VM, or VPS), configuring **LLM provider credentials** (Ollama Cloud API, local Ollama, OpenRouter), setting **execution guardrails**, configuring the **Gateway Service**, linking to **Discord**, and authenticating **GitHub CLI (`gh`)**.

---

## Tool Overview & Ecosystem Purpose

| Property | Details |
| :--- | :--- |
| **Tool Name** | **Hermes Agent** |
| **Tool Classification** | Open-source autonomous AI agent runtime and command execution engine developed by Nous Research. |
| **License Type** | Open Source (MIT License) |
| **Purpose in Ecosystem** | Functions as the primary autonomous developer and execution engine, receiving instructions via Discord (`hermes-bot`), communicating with LLMs (Ollama/OpenRouter), executing terminal scripts, parsing technical specifications, and updating GitHub repositories via `gh` CLI. |

---

## Overview & Architecture

Hermes Agent operates as an autonomous task execution engine. It accepts instructions, runs code and terminal commands, interacts with GitHub repositories via `gh`, and communicates with users and external workflow platforms (like **n8n**) through multi-channel gateways (such as **Discord**).

```
   +-------------------+        +----------------------+        +------------------------+
   |  Discord User /   |        |  Hermes Gateway      |        |  Hermes Agent Engine   |
   |  n8n Automation   | -----> |  Service Daemon      | -----> |  - LLM Provider       |
   |  (@hermes-bot)    |        |  (24/7 Service)      |        |  - Execution Guardrail |
   +-------------------+        +----------------------+        +------------------------+
                                                                            |
                                                                            v
                                                                +------------------------+
                                                                | Target Repositories    |
                                                                | GitHub CLI (`gh`)      |
                                                                +------------------------+
```

---

## Setup Archetypes & Deployment Modes

Before installation, choose the operational mode that fits your deployment architecture:

| Setup Archetype | Description | Primary Use Case | Execution Trigger |
| :--- | :--- | :--- | :--- |
| **1. Standalone CLI / Interactive Mode** | Runs directly in your terminal for single-turn or interactive conversation sessions. | Local development, debugging workflows, quick code generation tasks. | `hermes chat` or `hermes run "prompt"` |
| **2. Background Daemon Mode** | Operates as a local background daemon executing scheduled or queued local tasks. | Background task processing on local workstations or isolated VMs. | `hermes daemon start` |
| **3. Gateway Service Mode** *(Recommended)* | Connects Hermes to external messaging platforms (Discord, Telegram, Webhooks) to process multi-user inputs and automated workflow payloads. | Synergy Marketing Ecosystem bot integration for `n8n-bot` & Discord users. | `hermes gateway start` |
| **4. Containerized / VPS 24/7 Server Mode** | Deploys Hermes inside Docker or as a `systemd` daemon on a Virtual Private Server (VPS) with network isolation. | Enterprise production deployment ensuring 24/7 availability even when local PCs are off. | `systemctl enable --now hermes-gateway` |

- For local VM installation on Windows/Linux, see **[Oracle VirtualBox & Linux Mint Setup Guide](oracle-vm.md)**.
- For local VM installation on macOS, see **[UTM Virtualization Setup Guide for macOS](utm.md)**.

---

## Step 1: Installing Hermes Agent CLI

### Virtual Machine Setup Prerequisites (Ubuntu / Linux VM)

When setting up Hermes Agent inside a Virtual Machine (UTM, VirtualBox, or dedicated Linux VM), complete the following prerequisites first:

1. **Install `gedit`**: GUI text editor for easy config editing inside Linux VM.
   ```bash
   sudo apt update && sudo apt install gedit -y
   ```
2. **Install `curl`**: Essential package for running installer scripts.
   ```bash
   sudo apt install curl -y
   ```
3. **Install Ollama**: Required for running local models AND required for signing in to Ollama Cloud API via `ollama signin` / `ollama login`:
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ```
4. **Login on Discord in Browser** *(Optional)*:
   - Log in to [Discord](https://discord.com) in your VM's web browser to easily access the Developer Portal.
5. **Ready Setup of Discord Bots**:
   - Create your application at [Discord Developer Portal](https://discord.com/developers/applications).
   - Turn **ON** **Server Members Intent** and **Message Content Intent** (Bot settings).
   - Copy your **Bot Token** and numerical **Discord User ID** (from Discord Settings -> Advanced -> Developer Mode).

---

### Installing Hermes Agent

Open your terminal on your designated host environment and install the Hermes Agent package:

#### Linux / macOS / WSL2
```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

### Windows (PowerShell Administrator)
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

### Verification & Initial Setup
After installation completes, reload your terminal profile:
```bash
source ~/.bashrc
```

Verify installation version and launch the guided setup wizard:
```bash
hermes --version
hermes setup
```
*(Typing `hermes` or `hermes setup` starts the interactive configuration wizard to select your LLM provider and initialize configuration).*

---

## Step 2: Configuring LLM Providers & Models

Hermes requires access to a Large Language Model (LLM) provider. You can configure providers using the interactive CLI wizard, dedicated `hermes model` commands, editing environment variables (`.env`), or editing `config.yaml`.

---

### Option A: Managing Models via `hermes model` CLI Commands

The `hermes model` command suite allows you to add, list, switch, and test LLM models directly from your terminal.

#### 1. Add a Model (`hermes model add`)

- **Add Ollama Cloud API Model**:
  ```bash
  hermes model add \
    --name "gemma4:31b-cloud" \
    --provider "ollama" \
    --url "https://ollama.com/api" \
    --api-key "$OLLAMA_API_KEY"
  ```

- **Add Local Ollama Model**:
  1. Install Ollama locally (if not already installed):
     ```bash
     curl -fsSL https://ollama.com/install.sh | sh
     ```
  2. Download/pull your preferred model:
     ```bash
     ollama pull qwen3.6
     ```
  3. Register model with Hermes CLI (auto-detected at `http://127.0.0.1:11434`):
     ```bash
     hermes model add \
       --name "qwen3.6" \
       --provider "ollama" \
       --url "http://localhost:11434"
     ```

- **Add OpenRouter Model**:
  ```bash
  hermes model add \
    --name "meta-llama/llama-3.3-70b-instruct:free" \
    --provider "openrouter" \
    --api-key "$OPENROUTER_API_KEY"
  ```

> **Tip (Nous Portal):** You can also use **Nous Portal** (`hermes setup --portal`), a paid all-in-one subscription that bundles model access, web search, image generation, and other capabilities under one unified API key.

#### 2. List Configured Models (`hermes model list`)
```bash
hermes model list
```
*Output displays active model, provider, base URL, and health status.*

#### 3. Select Active Default Model (`hermes model set`)
```bash
hermes model set "gemma4:31b-cloud"
```

#### 4. Test Model Connection (`hermes model test`)
```bash
hermes model test "gemma4:31b-cloud"
```

---

### Option B: Modifying Settings via CLI `.env` Commands

You can set configuration parameters and environment variables non-interactively using CLI helper commands:

```bash
# Set LLM credentials
hermes env set OLLAMA_API_KEY="your_ollama_cloud_api_key_here"
hermes env set OPENROUTER_API_KEY="sk-or-v1-your-key-here"

# Set core LLM configuration
hermes config set llm.provider "ollama"
hermes config set llm.base_url "https://ollama.com/api"
hermes config set llm.model "gemma4:31b-cloud"

# View active environment configuration
hermes env list
```

---

### Option C: Manual `.env` File Modification

For headless environments or version-controlled deployment templates, you can create and edit the environment configuration file directly at `~/.config/hermes/.env` (or local project `.env`).

#### Editing via Terminal Text Editors

```bash
# Open or create .env using nano
nano ~/.config/hermes/.env
```
*(Or use `vim ~/.config/hermes/.env` or open in VS Code with `code ~/.config/hermes/.env`)*

#### Standard `.env` File Structure

Paste and edit the following template in your `.env` file:

```env
# ==============================================================================
# HERMES AGENT ENVIRONMENT CONFIGURATION
# Location: ~/.config/hermes/.env
# ==============================================================================

# LLM Provider Configuration (ollama | openrouter)
HERMES_LLM_PROVIDER="ollama"
HERMES_LLM_MODEL="gemma4:31b-cloud"
HERMES_OLLAMA_BASE_URL="https://ollama.com/api"
OLLAMA_API_KEY="your_ollama_cloud_api_key_here"
OPENROUTER_API_KEY="sk-or-v1-your-openrouter-key-here"

# Execution Guardrails (auto | semi-auto | manual)
HERMES_GUARDRAILS="auto"

# Gateway & Discord Integration
HERMES_GATEWAY_PLATFORM="discord"
HERMES_BOT_TOKEN="your_discord_bot_token_here"
HERMES_ALLOWED_USERS="123456789012345678,987654321098765432"

# GitHub Integration Credentials
GITHUB_TOKEN="ghp_your_github_personal_access_token_here"
```

After editing, restrict file permissions for security:
```bash
chmod 600 ~/.config/hermes/.env
```

---

### Option D: YAML Configuration (`~/.config/hermes/config.yaml`)

Alternatively, configure Hermes using the structured YAML file at `~/.config/hermes/config.yaml`:

```yaml
llm:
  provider: "ollama" # Options: ollama, openrouter
  base_url: "https://ollama.com/api" # Use http://localhost:11434 for local Ollama
  model: "gemma4:31b-cloud"
  api_key: "${OLLAMA_API_KEY}"

agent:
  system_prompt: "You are Hermes Agent, an autonomous marketing ecosystem developer integrated with Discord and GitHub."
  guardrails: "auto" # Options: auto, semi-auto, manual

gateway:
  platform: "discord"
  bot_token: "${HERMES_BOT_TOKEN}"
  allowed_users:
    - "123456789012345678" # Your numerical Discord User ID
```

---

## Step 3: Understanding & Configuring Execution Guardrails

Guardrails define the safety and autonomy level under which Hermes Agent executes terminal commands, writes code files, and makes repository commits.

### Guardrail Safety Modes

| Guardrail Mode | Description | Terminal Commands | File Creation / Edits | Git Commits & Pushes | Recommended Operating Environment |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`smart` / `auto` (Default)** | Intelligent autonomy. Hermes uses judgment to auto-approve safe read/list actions, block dangerous commands (e.g. deleting files, force pushing), and prompt for confirmation on ambiguous tasks. | Auto-approves safe; prompts on risky | Auto-approves safe; prompts on risky | Auto-approves safe; prompts on risky | Standard development, VM setup, and production daemon operation. |
| **`manual` (Strict)** | Maximum safety. Asks for explicit interactive user permission before performing *any* command execution, file read, file write, or external network call. | Prompts user | Prompts user | Prompts user | High-security environments or initial testing of unverified prompts. |
| **`off` (Autonomous)** | Disables approval guardrail prompts completely. Executes all requested actions without pausing. | Auto-approved | Auto-approved | Auto-approved | Isolated sandboxes, CI/CD runners, or trusted background tasks. |

### Configuring Guardrails

Choose your preferred configuration method:

- **Via CLI Command**:
  ```bash
  hermes config set agent.guardrails smart
  ```
- **Via `.env` File**:
  ```env
  HERMES_GUARDRAILS="smart"
  ```
- **Via `config.yaml` (`~/.hermes/config.yaml` or `~/.config/hermes/config.yaml`)**:
  ```yaml
  approvals:
    mode: smart # Options: smart, manual, off
    timeout: 300
  ```

> **Tip (Session Bypass):** During interactive terminal sessions, you can temporarily bypass approval prompts using the `/yolo` chat command (use with caution in non-isolated environments).

---

## Step 4: Setting Up & Managing the Gateway Service

The Gateway Service allows Hermes Agent to run as a 24/7 daemon that listens for incoming Discord messages, tags (e.g. `@hermes-bot`), and specification attachment files dispatched by **n8n** or team members.

---

### 1. Discord Bot Creation & Privileged Gateway Intents

Before starting the gateway, ensure your Discord Application in the [Discord Developer Portal](https://discord.com/developers/applications) is configured correctly:

1. **Authorization Flow**:
   - Set **Public Bot** to `ON`.
   - Leave **Require OAuth2 Code Grant** as `OFF`.
2. **Privileged Gateway Intents** *(Crucial Step)*:
   - Turn **ON**: **Server Members Intent**
   - Turn **ON**: **Message Content Intent**
   > ⚠️ **Important:** **Message Content Intent** is mandatory. If skipped, the bot will appear online on Discord but remain completely silent and fail to respond to any message.
3. **Bot Installation Scopes & Permissions**:
   - Scopes: `bot`, `applications.commands`
   - Permissions: `View Channels`, `Send Messages`, `Embed Links`, `Attach Files`, `Read Message History`.
4. **Access Protection**:
   - Hermes restricts bot interaction to authorized Discord User IDs only. Unlisted users are denied by default.

### 2. Interactive Gateway Setup (`hermes gateway setup`)

Run the wizard to configure your messaging provider:

```bash
hermes gateway setup
```

1. Select **Discord** as the gateway platform.
2. Enter your **Discord Bot Token** (`HERMES_BOT_TOKEN` created in [ref/discord.md](discord.md)).
3. Enter your numerical **Discord User ID** (obtained from Discord settings in Developer Mode) to populate the admin whitelist (`HERMES_ALLOWED_USERS`).
4. Confirm port and host settings (default gateway port: `8080`).

---

### 2. Gateway Lifecycle Commands

Manage the Gateway service manually from terminal:

```bash
# Start Gateway in foreground
hermes gateway start

# Check active Gateway status & healthy adapters
hermes gateway status

# Restart Gateway service
hermes gateway restart

# Stop Gateway service
hermes gateway stop
```

---

### 3. Installing Gateway as a 24/7 System Daemon

To ensure Hermes Gateway starts automatically on system boot and remains active 24/7 without keeping an interactive terminal session open:

#### Automated Service Installation (Linux / macOS)
```bash
hermes gateway install-service
```
*This command generates and registers a system service (`systemd` on Linux, `launchd` on macOS).*

#### Manual Linux `systemd` Unit Setup (Alternative)

Create `/etc/systemd/system/hermes-gateway.service`:

```ini
[Unit]
Description=Hermes Agent Gateway Service
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
EnvironmentFile=/home/ubuntu/.config/hermes/.env
ExecStart=/usr/local/bin/hermes gateway start
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now hermes-gateway
sudo systemctl status hermes-gateway
```

---

## Step 5: Installing & Authenticating GitHub CLI (`gh`)

Hermes Agent uses the GitHub CLI (`gh`) to clone user repositories, manage branches, and push updated site code inside `docs/` for GitHub Pages deployment.

### 1. Installation

#### **macOS (Homebrew)**
```bash
brew install gh
```

#### **Linux (Debian / Ubuntu)**
```bash
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

#### **Windows (winget)**
```powershell
winget install --id GitHub.cli
```

---

### 2. Authentication with GitHub PAT

Authenticate `gh` using the Personal Access Token created in [ref/github.md](github.md):

#### Option A: Interactive Login
```bash
gh auth login
```
- Select **GitHub.com**.
- Select **HTTPS** as preferred protocol.
- Authenticate Git with your GitHub credentials: **Yes**.
- Select **Paste an authentication token** and paste your Personal Access Token (PAT).

#### Option B: Non-Interactive Environment Setup
For automated headless server installations, authenticate directly via token and configure the Git credential helper:
```bash
export GITHUB_TOKEN="ghp_your_github_pat_classic_token_here"
echo "$GITHUB_TOKEN" | gh auth login --hostname github.com --git-protocol https --with-token
gh auth setup-git
```

> **Important Note:** Setting `GITHUB_TOKEN` as an environment variable alone is not enough for automated `git push` operations. Running `gh auth setup-git` sets up the Git credential helper so Hermes can push without interactive username/password prompts.

Verify authentication status:
```bash
gh auth status
```

---

### 3. Git Identity Signature Configuration

Configure global Git identity signatures so commits generated by Hermes bear clean metadata:

```bash
git config --global user.name "Hermes Agent Bot"
git config --global user.email "hermes-bot@users.noreply.github.com"
```

---

## Step 6: Complete End-to-End Hermes Agent Deployment Walkthrough

Follow this sequence for a clean, end-to-end installation of Hermes Agent using **Ollama Cloud API**, **Discord Gateway Integration**, **GitHub CLI Authentication**, and **24/7 Systemd Hosting**.

### Phase 1: Environment Preparation & Installation
```bash
# 1. Update host packages & install required VM tools (gedit, curl, git, jq)
sudo apt update && sudo apt install -y gedit curl git jq

# 2. Install Ollama (Required for local models and signing in to Ollama Cloud API via `ollama signin`)
curl -fsSL https://ollama.com/install.sh | sh

# 3. Install Hermes Agent CLI
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# 4. Reload terminal environment & verify
source ~/.bashrc
hermes --version
```

### Phase 2: Configure Ollama Cloud API & Add LLM Model
```bash
# 1. Set Ollama Cloud API Key in environment
export OLLAMA_API_KEY="your_ollama_cloud_api_key_here"
hermes env set OLLAMA_API_KEY="$OLLAMA_API_KEY"

# 2. Add Ollama Cloud API model via CLI
hermes model add \
  --name "gemma4:31b-cloud" \
  --provider "ollama" \
  --url "https://ollama.com/api" \
  --api-key "$OLLAMA_API_KEY"

# 3. Set active model and test endpoint connection
hermes model set "gemma4:31b-cloud"
hermes model test "gemma4:31b-cloud"
```

### Phase 3: Configure Guardrail Safety Controls
```bash
# Set guardrails to auto for autonomous gateway execution
hermes config set agent.guardrails auto
```

### Phase 4: Configure Discord Gateway Integration
```bash
# 1. Store Discord Bot Token and Admin User ID in .env
hermes env set HERMES_BOT_TOKEN="your_discord_bot_token_here"
hermes env set HERMES_ALLOWED_USERS="your_numerical_discord_user_id"

# 2. Run Gateway Setup Wizard
hermes gateway setup
```

### Phase 5: GitHub CLI (`gh`) Setup & Identity Config
```bash
# 1. Install GitHub CLI (Debian/Ubuntu)
sudo apt install gh -y

# 2. Authenticate non-interactively using PAT
export GITHUB_TOKEN="ghp_your_github_pat_token_here"
echo "$GITHUB_TOKEN" | gh auth login --with-token

# 3. Set global Git identity
git config --global user.name "Hermes Agent Bot"
git config --global user.email "hermes-bot@users.noreply.github.com"
```

### Phase 6: Enable 24/7 Gateway Daemon Service
```bash
# Install and start background system daemon
hermes gateway install-service
```

### Phase 7: Comprehensive System Diagnostic & Verification
```bash
# 1. Run Hermes Doctor diagnostic tool
hermes doctor

# 2. Run AI brain smoke test (verifies provider connection independent of Discord/Gateway)
hermes chat -q "Reply with exactly: provider ok"

# 3. Stream live Gateway logs (useful during troubleshooting)
tail -f ~/.hermes/logs/gateway.log
```

#### Discord Channel Verification
1. Open Discord and go to your server channel.
2. Send a test message tagging your bot:
   ```text
   @hermes-bot hello! Please report your status and active model.
   ```
3. Attach a test specification `.md` file to verify automated attachment parsing, repo cloning, `docs/` updates, and GitHub pushes.

---

## Step 7: Automated Provisioning & Setup Script

For headless servers, automated VM provisioning, or CI/CD infrastructure, you can execute this automated shell script to install, configure credentials, set LLM models, establish execution guardrails, authenticate GitHub CLI, and register the 24/7 Gateway service daemon in a single command.

### 1. Script Usage & Execution

Save the script as `setup-hermes.sh` or download it to your host machine:

```bash
# Make script executable
chmod +x setup-hermes.sh

# Option A: Run using environment variables passed inline
OLLAMA_API_KEY="your_api_key" \
HERMES_BOT_TOKEN="your_discord_token" \
HERMES_ALLOWED_USERS="123456789012345678" \
GITHUB_TOKEN="ghp_your_pat" \
./setup-hermes.sh

# Option B: Run by passing a path to a pre-populated .env file
./setup-hermes.sh ~/.config/hermes/.env
```

---

### 2. Complete Shell Provisioning Script (`setup-hermes.sh`)

```bash
#!/usr/bin/env bash
# ==============================================================================
# Automated Provisioning & Setup Script for Hermes Agent
# Ecosystem: Synergy Marketing Ecosystem
# Usage: ./setup-hermes.sh [path/to/env_file]
# ==============================================================================

set -e # Exit on error

# Terminal color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${BLUE}====================================================${NC}"
echo -e "${BLUE}=== Automated Hermes Agent Provisioning Engine ===${NC}"
echo -e "${BLUE}====================================================${NC}"

# 1. Load configuration from custom .env file if provided
ENV_FILE="${1:-$HOME/.config/hermes/.env}"

if [ -f "$ENV_FILE" ]; then
    echo -e "${GREEN}[+] Loading environment parameters from $ENV_FILE${NC}"
    export $(grep -v '^#' "$ENV_FILE" | xargs)
else
    echo -e "${YELLOW}[!] Config file $ENV_FILE not found. Utilizing environment variables.${NC}"
fi

# Configuration Defaults (overridden by loaded environment variables)
OLLAMA_API_KEY="${OLLAMA_API_KEY:-}"
OLLAMA_BASE_URL="${OLLAMA_BASE_URL:-https://ollama.com/api}"
HERMES_LLM_PROVIDER="${HERMES_LLM_PROVIDER:-ollama}"
HERMES_LLM_MODEL="${HERMES_LLM_MODEL:-gemma4:31b-cloud}"
HERMES_GUARDRAILS="${HERMES_GUARDRAILS:-auto}"
HERMES_BOT_TOKEN="${HERMES_BOT_TOKEN:-}"
HERMES_ALLOWED_USERS="${HERMES_ALLOWED_USERS:-}"
GITHUB_TOKEN="${GITHUB_TOKEN:-}"
GIT_USER_NAME="${GIT_USER_NAME:-Hermes Agent Bot}"
GIT_USER_EMAIL="${GIT_USER_EMAIL:-hermes-bot@users.noreply.github.com}"

# 2. Verify Host Prerequisites
echo -e "${GREEN}[+] Checking system prerequisites (curl, git, jq)...${NC}"
command -v curl >/dev/null 2>&1 || { echo -e "${RED}[!] curl is required but not installed. Aborting.${NC}"; exit 1; }
command -v git >/dev/null 2>&1 || { echo -e "${RED}[!] git is required but not installed. Aborting.${NC}"; exit 1; }

# 3. Install Hermes Agent CLI
if ! command -v hermes >/dev/null 2>&1; then
    echo -e "${GREEN}[+] Installing Hermes Agent CLI...${NC}"
    curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
    export PATH="$HOME/.hermes/bin:$PATH"
else
    echo -e "${GREEN}[+] Hermes Agent CLI is already installed.${NC}"
fi

# 4. Generate/Update ~/.config/hermes/.env File
echo -e "${GREEN}[+] Writing configuration file ~/.config/hermes/.env...${NC}"
mkdir -p "$HOME/.config/hermes"
cat <<EOF > "$HOME/.config/hermes/.env"
# Auto-generated by setup-hermes.sh
HERMES_LLM_PROVIDER="${HERMES_LLM_PROVIDER}"
HERMES_LLM_MODEL="${HERMES_LLM_MODEL}"
HERMES_OLLAMA_BASE_URL="${OLLAMA_BASE_URL}"
OLLAMA_API_KEY="${OLLAMA_API_KEY}"
HERMES_GUARDRAILS="${HERMES_GUARDRAILS}"
HERMES_GATEWAY_PLATFORM="discord"
HERMES_BOT_TOKEN="${HERMES_BOT_TOKEN}"
HERMES_ALLOWED_USERS="${HERMES_ALLOWED_USERS}"
GITHUB_TOKEN="${GITHUB_TOKEN}"
EOF
chmod 600 "$HOME/.config/hermes/.env"

# 5. Configure LLM Model & Guardrails via Hermes CLI
echo -e "${GREEN}[+] Setting active model and guardrails...${NC}"
if [ -n "$OLLAMA_API_KEY" ]; then
    hermes model add \
        --name "$HERMES_LLM_MODEL" \
        --provider "$HERMES_LLM_PROVIDER" \
        --url "$OLLAMA_BASE_URL" \
        --api-key "$OLLAMA_API_KEY" || true
fi

hermes model set "$HERMES_LLM_MODEL" || true
hermes config set agent.guardrails "$HERMES_GUARDRAILS" || true

# 6. Install & Authenticate GitHub CLI (`gh`)
echo -e "${GREEN}[+] Setting up GitHub CLI and Git identity...${NC}"
if ! command -v gh >/dev/null 2>&1; then
    echo -e "${YELLOW}[!] Installing gh CLI package...${NC}"
    sudo apt update && sudo apt install -y gh || true
fi

if [ -n "$GITHUB_TOKEN" ]; then
    echo "$GITHUB_TOKEN" | gh auth login --with-token || true
fi

git config --global user.name "$GIT_USER_NAME"
git config --global user.email "$GIT_USER_EMAIL"

# 7. Setup Discord Gateway Service & Install Background Daemon
echo -e "${GREEN}[+] Configuring Discord Gateway and installing system daemon...${NC}"
if [ -n "$HERMES_BOT_TOKEN" ]; then
    hermes gateway setup --non-interactive \
        --platform discord \
        --bot-token "$HERMES_BOT_TOKEN" \
        --allowed-users "$HERMES_ALLOWED_USERS" || true

    hermes gateway install-service || true
fi

# 8. System Diagnostic Verification
echo -e "${BLUE}=== Executing System Health Check (hermes doctor) ===${NC}"
hermes doctor || true

echo -e "${GREEN}====================================================${NC}"
echo -e "${GREEN}=== Automated Provisioning Completed Successfully! ===${NC}"
echo -e "${GREEN}====================================================${NC}"
```

---

## Security & Operational Guidelines

> [!WARNING]
> **24/7 Host Availability**
> If your host machine or Virtual Machine is powered off, Hermes Agent will cease responding on Discord. For reliable production operations, deploy Hermes on a dedicated 24/7 Virtual Private Server (VPS) or cloud VM.

> [!IMPORTANT]
> **Privileged Terminal Execution & User ID Whitelisting**
> Because Hermes Agent runs shell commands, keep `HERMES_ALLOWED_USERS` strictly restricted to trusted numerical Discord User IDs. Unauthorized users must not be permitted to issue execution prompts.

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| `hermes doctor` reports LLM endpoint connection error | Selected provider (Ollama Cloud, local Ollama, or OpenRouter) is unreachable or missing API key. | Test model connection using `hermes model test <model_name>` or smoke test `hermes chat -q "Reply with exactly: provider ok"`. Verify API keys in `~/.config/hermes/.env` or `~/.hermes/.env`. |
| Discord bot is online but never replies | **Message Content Intent** is disabled in Discord Developer Portal. | Go to Discord Developer Portal > Bot > Privileged Gateway Intents > enable **Message Content Intent** > Save > restart gateway (`hermes gateway restart`). |
| Discord bot ignores specific user | Requesting Discord User ID is missing from `HERMES_ALLOWED_USERS`. | Add your numerical Discord User ID to `HERMES_ALLOWED_USERS` in `.env` and restart gateway. |
| "Disallowed Intents" error on gateway startup | Privileged intents not enabled in Developer Portal. | Enable Presence, Server Members, and Message Content intents in Discord Developer Portal. |
| `git push` prompts for username/password | Git credential helper not configured on host environment. | Run `sudo apt install gh -y`, `echo "$GITHUB_TOKEN" | gh auth login --hostname github.com --git-protocol https --with-token`, and `gh auth setup-git`. |
| Hermes asks to approve commands repeatedly | Expected behavior under `smart` / `manual` guardrail safety modes. | Approve prompts with `once`/`session`/`always`, or type `/yolo` in chat to temporarily bypass approvals. |
| `hermes model add` fails with invalid endpoint | Base URL syntax error or missing API path (`/api`). | Ensure Ollama Cloud URL is `https://ollama.com/api` and local Ollama is `http://localhost:11434`. |
| `.env` variables fail to load or throw syntax error | Syntax error in configuration file (unquoted spaces or missing `=` sign). | Inspect `.env` file for typos. Ensure values with special characters are wrapped in double quotes `"..."`. |
| Gateway daemon fails to launch on system boot | Systemd service file path incorrect or missing environment variable file. | Check service logs using `journalctl -u hermes-gateway -e` or inspect `tail -f ~/.hermes/logs/gateway.log`. |
| Local scratch directories accumulating disk space | Post-execution scratch folder cleanup did not trigger after push. | Run `hermes clean` in terminal to prune temporary scratch workspace directories. |

---

## Related Documents
- [Overview Page](../overview.md)
- [Ollama Setup & Configuration Guide](ollama.md)
- [Discord Bot Creation & Gateway Guide](discord.md)
- [GitHub PAT & Pages Setup Guide](github.md)
- Next Step: [Google Account & Cloud Console Setup](google-account.md)

