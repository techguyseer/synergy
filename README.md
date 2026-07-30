## Repository Structure

```text
synergy/
├── README.md               # Quick overview & repository index
├── .gitignore              # Git ignore configuration (excludes workflow/)
├── overview.md             # Complete step-by-step setup guide & lab checklist
├── ref/                    # Core integration & setup reference guides
│   ├── ollama.md           # Local & cloud Ollama LLM setup
│   ├── openrouter.md       # OpenRouter model routing & API key config
│   ├── google-account.md   # Google Cloud Console & OAuth 2.0 credentials
│   ├── discord.md          # Discord developer portal & dual-bot setup
│   ├── github.md           # GitHub Personal Access Tokens & Pages hosting
│   ├── hermes.md           # Hermes Agent installation & gateway setup
│   ├── n8n.md              # Self-hosting n8n (Native & Docker)
│   ├── oracle-vm.md        # Oracle VirtualBox & Linux Mint VM guide
│   └── utm.md              # UTM Virtualization setup for macOS
└── docs/                   # GitHub Pages documentation portal
    └── index.html
```

---

## Setup Guides & Reference Index

| Guide | Description |
| :--- | :--- |
| **[Master Setup Guide](overview.md)** | Full laboratory setup guide, architecture details, and checklist |
| **[Ollama Guide](ref/ollama.md)** | Installing Ollama locally and connecting n8n to local LLMs |
| **[OpenRouter Guide](ref/openrouter.md)** | Setting up OpenRouter API keys and fallback model chains |
| **[Google Account Guide](ref/google-account.md)** | Google Cloud OAuth 2.0 setup for Gmail, Docs, and Drive APIs |
| **[Discord Dual-Bot Guide](ref/discord.md)** | Configuring `n8n-bot` and `hermes-bot` with Discord Gateway |
| **[GitHub & Pages Guide](ref/github.md)** | Setting up GitHub Personal Access Tokens and configuring GitHub Pages hosting |
| **[Hermes Agent Guide](ref/hermes.md)** | Installing Hermes Agent and linking LLMs, Discord, & GitHub CLI |
| **[n8n Deployment Guide](ref/n8n.md)** | Self-hosting n8n natively or via Docker Compose |
| **[VirtualBox Setup](ref/oracle-vm.md)** | Provisioning Linux Mint VMs for isolated agent execution |
| **[UTM Setup for Mac](ref/utm.md)** | Setting up UTM virtual machines on macOS host systems |

---

## Quick Start

1. Check out the **[Master Setup Guide (`overview.md`)](overview.md)** for a complete walkthrough.
2. Import the n8n workflows into your n8n instance.
3. Follow the guides in **[`/ref`](ref/)** to configure API credentials for Google Cloud, Discord, GitHub, and your chosen LLM provider.