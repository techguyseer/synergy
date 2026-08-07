# Ollama Setup & Configuration Guide

This guide covers setting up **Ollama** locally on your desktop environment and configuring **Ollama Cloud** services for remote/cloud model access. This is a prerequisite component of the **Synergy Marketing Ecosystem**.

---

## Tool Overview & Ecosystem Purpose

| Property | Details |
| :--- | :--- |
| **Tool Name** | **Ollama & Ollama Cloud** |
| **Tool Classification** | Open-source LLM inference framework, local model server, and cloud model API service. |
| **License Type** | Open Source (MIT License for Ollama server; commercial/paid tier for Ollama Cloud API) |
| **Purpose in Ecosystem** | Provides local or cloud-hosted open-source AI intelligence (e.g., Qwen, Llama, Gemma) powering the reasoning capability for both n8n workflow nodes and Hermes Agent execution sessions. |

---

## Part 1: Downloading & Running Ollama Locally

Ollama allows you to run open-source large language models (LLMs) locally on your own machine.

### 1. Installation per Operating System

#### **macOS**
1. Download the macOS application installer from the official website: [ollama.com/download/mac](https://ollama.com/download/mac).
2. Unzip the downloaded archive and drag `Ollama.app` into your `Applications` folder.
3. Open Ollama from Applications. It will prompt you to install command-line tools in your terminal automatically.

#### **Windows**
1. Download the Windows installer from [ollama.com/download/windows](https://ollama.com/download/windows).
2. Run `OllamaSetup.exe` and follow the setup wizard prompts.
3. Once completed, Ollama will run in the background (visible in your system tray).

#### **Linux**
Install Ollama via terminal using the official automated installation script:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

---

### 2. Running Models Locally

Once Ollama is installed, verify the service is active by opening your terminal or command prompt:

```bash
# Check installed version
ollama --version
```

#### **Discovering Supported Models**
You can browse the complete list of officially supported open-source models, parameter sizes (e.g., 7b, 8b, 14b, 70b), and model tags on the official Ollama Model Library:
- **Official Model Library**: [ollama.com/library](https://ollama.com/library)

Popular supported models include:
- **Llama 3.3 / Llama 3**: Meta's state-of-the-art open models (`ollama run llama3.3` or `ollama run llama3`)
- **DeepSeek-R1**: Open-weights reasoning models (`ollama run deepseek-r1`)
- **Qwen 2.5**: Alibaba Cloud's multilingual and coding models (`ollama run qwen2.5`)
- **Mistral / Mixtral**: High-performance general models (`ollama run mistral`)
- **Gemma 2**: Google's lightweight open model family (`ollama run gemma2`)

#### **Pulling and Running a Model**
To start interacting with a model, use the `ollama run` command. Ollama will automatically download the model if it is not present locally:

```bash
# Run Llama 3 (8B parameter model)
ollama run llama3

# Run Mistral
ollama run mistral

# Run Qwen 2.5
ollama run qwen2.5
```

#### **Useful CLI Commands**
```bash
# List all locally downloaded models
ollama list

# Remove a model to free disk space
ollama rm llama3

# Check active running models in memory
ollama ps
```

---

### 3. Configuring Local Network Access for n8n & Agents

By default, Ollama binds to `127.0.0.1:11434` (localhost). If **n8n** or **Hermes Agent** is running inside Docker containers or on another local network device, Ollama needs to listen on all interfaces (`0.0.0.0`).

#### **Setting `OLLAMA_HOST` Environment Variable**:
- **macOS / Linux**:
  ```bash
  export OLLAMA_HOST="0.0.0.0:11434"
  ```
- **Windows**:
  Set an Environment Variable via System Properties:
  `Variable Name: OLLAMA_HOST`, `Variable Value: 0.0.0.0:11434`.

#### **Docker Container DNS Resolution (`host.docker.internal`)**:
When n8n or an agent is hosted inside a Docker container while Ollama runs directly on the host operating system:
- **macOS / Windows Docker Desktop**: Use `http://host.docker.internal:11434` as the host base URL inside n8n.
- **Linux Docker**: Pass `--add-host=host.docker.internal:host-gateway` in your `docker run` command or `extra_hosts` in `docker-compose.yml`, then target `http://host.docker.internal:11434`.
- **Docker Compose (Same Network)**: If Ollama itself is deployed as a container alongside n8n in the same Docker network, use the container service name, e.g., `http://ollama:11434`.
- For complete n8n self-hosting and Docker Compose instructions, see **[n8n Self-Hosting & Deployment Guide](n8n.md)**.

> **Warning: Network Security Warning**
> Exposing Ollama to `0.0.0.0` allows any device on your local network to access your local LLMs without authentication. Ensure your network/router firewall restricts public access to port `11434`.

---

## Part 2: Base URLs & Endpoints Reference for n8n Integrations

When connecting n8n Ollama nodes, Hermes Agent, or HTTP request nodes, use the appropriate base URL according to your deployment architecture:

| Environment / Deployment Architecture | Recommended Base URL | Context & Usage |
| :--- | :--- | :--- |
| **Local Host / Non-Docker** | `http://localhost:11434` or `http://127.0.0.1:11434` | n8n and Ollama are running natively on the same machine. |
| **n8n inside Docker to Ollama on Host Machine** | `http://host.docker.internal:11434` | n8n container connects to Ollama running on the host OS. |
| **n8n and Ollama in same Docker Compose Network** | `http://ollama:11434` | Both n8n and Ollama are containerized services on the same internal Docker network. |
| **Remote Host / VPS** | `http://<YOUR_SERVER_IP>:11434` | n8n connects to a remote server running Ollama (`OLLAMA_HOST=0.0.0.0`). |
| **Ollama Cloud Service** | `https://ollama.com` or `https://ollama.com/api` | n8n or agents invoking hosted Ollama Cloud models via API key authentication. |

### Configuring Ollama Credentials in n8n

In your n8n workflow nodes (such as the Ollama Chat Model node `@n8n/n8n-nodes-langchain.lmChatOllama` used in Marketing Agent, Orchestrator, and Analytics Agent):

1. Go to **n8n > Credentials > Add Credential** and search for `Ollama API`.
2. Name the credential `Ollama account` (or your preferred name).
3. Set the **Host** parameter according to your deployment table above (e.g., `http://host.docker.internal:11434`).
4. Select your target model (specifically `gemma4:31b-cloud` as the ecosystem model of choice, or `llama3` / `qwen2.5`) inside the Ollama Chat Model node settings.

---

## Part 3: Ollama Cloud Subscription & API Key Setup

For larger models or instances where local GPU hardware is insufficient, Ollama Cloud provides managed cloud endpoint execution.

### 1. Creating an Account & Subscribing
1. Navigate to [ollama.com](https://ollama.com) and click **Sign Up** or **Sign In**.
2. Complete your profile setup and select a cloud tier or pay-as-you-go subscription plan if required.

---

### 2. Signing In via Terminal (`ollama signin`)

To authenticate your local CLI environment with your Ollama Cloud account:

1. Open your terminal and run:
   ```bash
   ollama signin
   ```
2. Follow the browser prompt or authentication code to approve device access.
3. Once authenticated, your local CLI can invoke cloud-hosted models (e.g. `ollama run <model-name>:cloud`).

---

### 3. Generating & Managing API Keys

When connecting **n8n nodes** or custom scripts to Ollama Cloud, authentication is performed via API keys.

1. Go to your account settings: [ollama.com/settings/keys](https://ollama.com/settings/keys).
2. Click **Generate New Key**.
3. Name your key (e.g., `n8n-workflow-agent`).
4. **Copy the API key immediately** and store it in a password manager.

#### **Using API Keys in Environment Variables & Requests**
To set the key globally on your agent system:

```bash
export OLLAMA_API_KEY="your_secret_api_key_here"
```

In HTTP requests (or n8n custom HTTP nodes):
```bash
curl https://ollama.com/api/chat \
  -H "Authorization: Bearer $OLLAMA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma4:31b-cloud",
    "messages": [{"role": "user", "content": "Hello world"}]
  }'
```

---

### Security & Operational Guidelines

> **Important: API Key Storage**
> - Treat your `OLLAMA_API_KEY` with the same security level as a password.
> - Never hardcode API keys directly inside n8n workflow export JSON files or public repositories. Use n8n's credential manager or environment variables.

> **Note: Cloud vs. Local Execution**
> - Local execution requires zero internet connection after pulling models, making it ideal for offline processing and complete data privacy.
> - Cloud execution requires active internet access and valid authentication headers.

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| `ollama run` returns `connection refused` or `could not connect` | Ollama background service daemon is not active. | Open `Ollama.app` (macOS), check Windows system tray, or run `sudo systemctl start ollama` (Linux). |
| Extremely slow generation speed / host system lag | Selected model size (e.g. 70B) exceeds GPU VRAM, swapping layers to system RAM/disk. | Switch to smaller 7B/8B models: `ollama run qwen2.5:7b` or `ollama run llama3.2:3b`. |
| n8n or remote machine cannot reach Ollama on port 11434 | Ollama host service is bound exclusively to `127.0.0.1`. | Set environment variable `OLLAMA_HOST="0.0.0.0:11434"` on host machine and restart Ollama service. |
| `CUDA out of memory` / GPU fallback during execution | Model context window size or parallel request count exceeds VRAM capacity. | Reduce context window setting (`num_ctx 4096`) or restrict parallel instances via `OLLAMA_NUM_PARALLEL=1`. |
| `ollama pull` fails with `digest mismatch` / network timeout | Unstable network connection during multi-GB model download. | Re-run `ollama pull <model-name>`. Ollama automatically resumes incomplete chunk downloads. |
| Ollama Cloud returns `401 Unauthorized` | Missing or expired `OLLAMA_API_KEY` header on HTTP requests. | Generate a new key at [ollama.com/settings/keys](https://ollama.com/settings/keys) and pass `Authorization: Bearer $OLLAMA_API_KEY`. |
| Linux `systemctl` fails with `ollama.service: Failed with result 'exit-code'` | Systemd unit file lacks proper directory permissions or GPU driver path. | Check `journalctl -u ollama -e` to inspect driver issues, and ensure `/usr/local/bin/ollama` has execution rights. |

---

## Related Documents
- [Overview Page](../overview.md)
- [OpenRouter Setup & Model Routing Guide](openrouter.md)
- Next Step: [Google Account & Cloud Console Setup](google-account.md)



