# n8n Self-Hosting & Deployment Guide

This guide details how to self-host and run an independent **n8n** instance using either **Native NPM/npx** or **Docker / Docker Compose** within the **Synergy Marketing Ecosystem** infrastructure options.

---

## Part 1: Overview & System Requirements

n8n is an open-source workflow automation platform. Self-hosting n8n provides full control over data privacy, execution timeouts, custom credentials, and local network connectivity to LLM servers like Ollama.

### Minimum System Requirements
- **CPU**: 1 vCPU (2+ cores recommended for heavy workflows)
- **RAM**: 1 GB minimum (2 GB+ recommended)
- **Disk**: 10 GB SSD storage
- **Node.js**: v18.x or v20.x (for native installations)

---

## Part 2: Native Hosting (npm / npx)

Native hosting runs n8n directly on your host operating system using Node.js.

### 1. Prerequisites
Ensure Node.js and npm are installed:
```bash
# Verify Node.js version (v18 or v20 required)
node -v
npm -v
```

### 2. Global Installation & Launch
Install n8n globally via npm:
```bash
npm install n8n -g
```

To launch n8n on port `5678`:
```bash
n8n start
```

For quick temporary testing without global installation:
```bash
npx n8n
```

### 3. Accessing the Web Interface
Open your browser and navigate to:
```text
http://localhost:5678
```
On first launch, create your owner user account to secure the instance.

---

## Part 3: Docker & Docker Compose Hosting (Recommended)

Running n8n in Docker isolates runtime dependencies and simplifies upgrades and data volume persistence.

### 1. Standalone Docker Container

To run n8n in a standalone container with persistent storage:

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_HOST="localhost" \
  -e N8N_PORT="5678" \
  -e N8N_PROTOCOL="http" \
  -e NODE_ENV="production" \
  -e WEBHOOK_URL="http://localhost:5678/" \
  docker.n8n.io/n8nio/n8n:latest
```

### 2. Docker Compose (with Environment Variables)

Create a directory `n8n-docker` and add the following files:

#### `docker-compose.yml`
```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-selfhosted
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST:-localhost}
      - N8N_PORT=5678
      - N8N_PROTOCOL=${N8N_PROTOCOL:-http}
      - NODE_ENV=production
      - WEBHOOK_URL=${WEBHOOK_URL:-http://localhost:5678/}
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE:-UTC}
    volumes:
      - n8n_storage:/home/node/.n8n

volumes:
  n8n_storage:
```

#### Launching the Stack
```bash
# Start container in detached mode
docker compose up -d

# Check running status and logs
docker compose ps
docker compose logs -f n8n
```

---

## Part 4: Environment Variables & Network Configuration

### 1. Essential Environment Variables

| Variable | Default | Purpose |
| :--- | :--- | :--- |
| `N8N_HOST` | `localhost` | Domain name or IP address of your n8n host |
| `N8N_PORT` | `5678` | Listening network port |
| `N8N_PROTOCOL` | `http` | Network protocol (`http` or `https`) |
| `WEBHOOK_URL` | `http://localhost:5678/` | Base URL used for external webhook triggers |
| `EXECUTIONS_DATA_PRUNE` | `true` | Automatically delete old execution history |
| `EXECUTIONS_DATA_MAX_AGE` | `168` | Prune execution logs older than X hours (168h = 7 days) |

### 2. Connecting Dockerized n8n to Local Ollama
When n8n runs inside Docker and Ollama runs on your host machine:
- Use `http://host.docker.internal:11434` as the Ollama Base URL.
- On Linux Docker hosts, ensure `extra_hosts: ["host.docker.internal:host-gateway"]` is added to your `docker-compose.yml`.

---

## Part 5: Maintenance & Upgrades

### Upgrading Docker Instance
```bash
docker compose pull
docker compose up -d --remove-orphans
```

### Upgrading Native npm Instance
```bash
npm install n8n@latest -g
```
