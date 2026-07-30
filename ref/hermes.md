# Hermes Agent Installation & Integration Guide

This guide provides step-by-step instructions for installing the **Hermes Agent** as the autonomous execution layer of the **Synergy Marketing Ecosystem** — running on a separate environment (local machine, WSL2, or VPS), configuring **LLM provider credentials**, linking to **Discord**, and installing/authenticating **GitHub CLI (`gh`)**.

---

## Overview

Hermes Agent runs as an autonomous agent daemon on a designated host environment. It communicates with users via Discord, leverages LLMs (Ollama or OpenRouter) for processing, and interacts with GitHub repositories via the GitHub CLI.

---

## Step 1: Target Environment & Installation

Hermes Agent can run directly on your host operating system, inside WSL2, or within a dedicated isolated Virtual Machine:
- **Isolated VM on Windows/Linux**: Follow **[Oracle VirtualBox & Linux Mint Setup Guide](oracle-vm.md)**.
- **Isolated VM on macOS**: Follow **[UTM Virtualization Setup Guide for macOS](utm.md)**.

### Installation Command (Linux / macOS / WSL2)
Open your terminal and run:
```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

### Windows (PowerShell)
Open PowerShell as Administrator and run:
```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

### Verification
After installation completes, restart your terminal session and verify the installation:
```bash
hermes --version
```

---

## Step 2: Configuring LLM Providers (Ollama / OpenRouter)

Hermes requires access to a Large Language Model provider to process agent instructions.

### Option A: Interactive CLI Setup
Run the built-in configuration wizard:
```bash
hermes setup
```
1. Select your preferred LLM provider:
   - **Ollama (Local/Cloud)**: Specify your Ollama endpoint URL (e.g., `http://localhost:11434` or remote IP `http://192.168.1.100:11434`) and model name (e.g., `llama3` or `qwen2.5`). See [ref/ollama.md](ollama.md).
   - **OpenRouter**: Enter your OpenRouter API Key (`sk-or-v1-...`). For complete setup instructions, model selection (`:free` vs paid), and auto-routing features, see [ref/openrouter.md](openrouter.md).
2. Save the configuration.

### Option B: Manual Configuration Files
Alternatively, you can edit configuration files or set environment variables directly on your system.

Create or update the configuration file at `~/.config/hermes/config.yaml` (or environment variables):

```yaml
llm:
  provider: "ollama" # Options: ollama, openrouter
  base_url: "http://127.0.0.1:11434"
  model: "llama3"
  api_key: "" # Leave empty for local Ollama, or insert OLLAMA_API_KEY / OpenRouter Key

agent:
  system_prompt: "You are an AI assistant connected via Discord and integrated with GitHub workflows."
```

Or set environment variables in your shell configuration (`~/.bashrc` or `~/.zshrc`):
```bash
export HERMES_LLM_PROVIDER="ollama"
export HERMES_OLLAMA_BASE_URL="http://127.0.0.1:11434"
export OPENROUTER_API_KEY="your_openrouter_api_key_here"
```

---

## Step 3: Integrating Hermes Agent with Discord

Link your running Hermes Agent to the Discord Bot created in the [Discord Setup Guide](ref/discord.md).

1. Execute the gateway setup command in your terminal:
   ```bash
   hermes gateway setup
   ```
2. Select **Discord** as the gateway platform.
3. Enter your **Discord Bot Token** (`HERMES_BOT_TOKEN` for `hermes-bot`) when prompted.
4. Enter your numerical **Discord User ID** (obtained from Discord settings in Developer Mode).
5. Start the gateway service:
   ```bash
   hermes gateway start
   ```

### Verifying Gateway Connection
Run the diagnostic check to ensure all components are healthy:
```bash
hermes doctor
```
Once active, send a Direct Message to `hermes-bot` or tag `@hermes-bot hello` in your Discord server channel to verify active communication.

### Automated Specification Execution & Repository Workflow
When `n8n-bot` dispatches a website or analytics build task, it tags `@hermes-bot` in your Discord channel with an attached technical specification `.md` file.

Hermes Agent automatically executes the following sequence:
1. **Reads Attachment**: Downloads and parses the attached specification (`website-specification.md` or `analytics-specification.md`).
2. **Clones Custom Repository**: Clones the user's custom GitHub repository URL specified in the n8n prompt (e.g., `https://github.com/<your-username>/website-prototype.git`).
3. **Builds Site Assets in `/docs`**: Generates or updates static HTML5, CSS3, and JavaScript files inside the repository's `docs/` directory.
4. **Commits & Pushes**: Commits the changes with Git and pushes to the `main` branch, triggering GitHub Pages deployment.
5. **Scratch Cleanup**: Deletes local source code scratch directories after pushing to ensure a clean workspace for future requests.
6. **Discord Status Reply**: Sends a short, concise status confirmation reply back to the requestor on Discord.

---

## Step 4: Installing & Authenticating GitHub CLI (`gh`)

To allow Hermes Agent to clone repositories, create pull requests, push code changes, and manage GitHub Pages, install and authenticate the **GitHub CLI (`gh`)**.

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

Authenticate `gh` using the Personal Access Token created in the [GitHub Setup Guide](ref/github.md):

#### Option A: Interactive Login
```bash
gh auth login
```
- Select **GitHub.com**.
- Select **HTTPS** as preferred protocol for Git operations.
- Authenticate Git with your GitHub credentials: **Yes**.
- Select **Paste an authentication token** and paste your Personal Access Token (PAT).

#### Option B: Non-Interactive Environment Variable Setup
For automated/headless server environments, pass the PAT via environment variable:
```bash
export GITHUB_TOKEN="your_github_pat_classic_token_here"
echo "$GITHUB_TOKEN" | gh auth login --with-token
```

---

### 3. Git Identity Configuration

Set standard Git user signatures so commits pushed by Hermes Agent bear clean metadata:

```bash
git config --global user.name "Hermes Agent Bot"
git config --global user.email "hermes-bot@users.noreply.github.com"
```

---

## Security & Operational Guidelines

> **Warning: 24/7 Hosting & Execution Isolation**
> If your workstation is turned off, Hermes Agent will cease responding on Discord. To maintain 24/7 availability, run Hermes on a dedicated Virtual Private Server (VPS) or cloud instance (e.g., Ubuntu LTS).

> **Important: Privileged Command Execution**
> Hermes Agent possesses code execution capabilities. Restrict Discord User ID access strictly to trusted administrators to prevent unauthorized users from issuing malicious terminal commands through Discord.

---

## Related Documents
- [Overview Page](file:///Users/seerneil/Documents/codespaces/ailab/synergy/overview.md)
- Previous Step: [GitHub PAT & GitHub Pages Guide](ref/github.md)
