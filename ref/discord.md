# Discord Developer & Dual-Bot Integration Guide

This document outlines the step-by-step process for creating **two separate Discord Developer Applications and Bots** for the **Synergy Marketing Ecosystem** — one named **`n8n-bot`** for n8n Workflows and one named **`hermes-bot`** for Hermes Agent.

---

## Overview & Dual-Bot Architecture

To maintain security, clear event boundaries, and isolated token management, this ecosystem utilizes two distinct Discord bots:

1. **`n8n-bot`**: Dedicated to n8n workflow triggers, scheduled channel notifications, embeds, and webhooks.
2. **`hermes-bot`**: Dedicated to interactive AI chat, prompt execution, direct message sessions, and command routing.

---

## Step 1: Create Discord Developer Applications

You must repeat the application creation process twice in the [Discord Developer Portal](https://discord.com/developers/applications).

### Application 1: `n8n-bot`
1. Open the [Discord Developer Portal](https://discord.com/developers/applications).
2. Click **New Application**.
3. Name: `n8n-bot`.
4. Accept terms and click **Create**.

### Application 2: `hermes-bot`
1. Click **Applications** in the top menu to return to your apps list.
2. Click **New Application**.
3. Name: `hermes-bot`.
4. Accept terms and click **Create**.

---

## Step 2: Configure Bots & Obtain Bot Tokens

Perform these steps for **both** applications:

### 1. Configure `n8n-bot`
1. Select the `n8n-bot` application.
2. Navigate to the **Bot** tab in the left sidebar.
3. Under **Build-A-Bot**, click **Reset Token** to generate a secret token.
4. Copy and label this token securely: `N8N_BOT_TOKEN`.

### 2. Configure `hermes-bot`
1. Select the `hermes-bot` application.
2. Navigate to the **Bot** tab in the left sidebar.
3. Under **Build-A-Bot**, click **Reset Token** to generate a secret token.
4. Copy and label this token securely: `HERMES_BOT_TOKEN`.

> **Warning: Token Security Notice**
> Do not interchange or expose these tokens. Discord actively scans public GitHub repositories and automatically revokes leaked bot tokens immediately.

---

## Step 3: Enable Privileged Gateway Intents

Enable the required gateway intents for both bots on their respective **Bot** settings pages.

1. On **n8n-bot > Bot**:
   - Scroll to **Privileged Gateway Intents**.
   - Enable **Message Content Intent**.
   - Click **Save Changes**.

2. On **hermes-bot > Bot**:
   - Scroll to **Privileged Gateway Intents**.
   - Enable **Message Content Intent** (Mandatory for Hermes to read user prompts).
   - Enable **Server Members Intent** (Optional for member resolution).
   - Click **Save Changes**.

---

## Step 4: Generate Invite URLs & Add Both Bots to Discord Server

Generate separate invite links for each bot via **OAuth2 > URL Generator**.

### 1. Invite `n8n-bot`
1. Go to **n8n-bot > OAuth2 > URL Generator**.
2. Select Scope: `bot`.
3. Select Permissions:
   - **View Channels / Read Messages**
   - **Send Messages**
   - **Embed Links**
   - **Attach Files**
4. Copy the generated URL, open it in your browser, select your server, and click **Authorize**.

### 2. Invite `hermes-bot`
1. Go to **hermes-bot > OAuth2 > URL Generator**.
2. Select Scope: `bot`.
3. Select Permissions:
   - **View Channels / Read Messages**
   - **Send Messages**
   - **Read Message History**
   - **Embed Links**
   - **Attach Files**
   - **Add Reactions**
4. Copy the generated URL, open it in your browser, select your server, and click **Authorize**.

---

## Step 5: Obtain Discord User & Channel IDs

For the laboratory workflows (Development Agent and Analytics Agent), you must copy your Discord User ID and target Channel ID.

### 1. Obtain Your Discord User ID (For Hermes Authorization)
1. Open Discord, go to **User Settings (gear icon) > Advanced**.
2. Toggle **Developer Mode** to **ON**.
3. Right-click your profile avatar/name in Discord and click **Copy User ID**.
4. Save this numerical ID (e.g., `123456789012345678`) for Hermes configuration.

### 2. Obtain Your Target Channel ID & Guild ID
1. In your Discord server, right-click the text channel designated for agent communications (e.g., `#agent-lab`).
2. Click **Copy Channel ID** and save the ID.
3. Right-click your Server icon in the server list and click **Copy Server ID** (Guild ID).
4. In n8n, update the `Send a message to Hermes` node in the Development Agent and Analytics Agent workflows with your Server ID and Channel ID.

---

## Step 6: Connecting Bots & Understanding the Mention Protocol

### 1. Connecting `n8n-bot` to n8n
1. Open **n8n Web Interface**.
2. Navigate to **Credentials > Add Credential** and select `Discord Bot API`.
3. Paste the `N8N_BOT_TOKEN` and save as `Discord Bot account`.

### 2. Connecting `hermes-bot` to Hermes Agent
1. In your Hermes Agent terminal environment, run:
   ```bash
   hermes gateway setup
   ```
2. Select **Discord** as the gateway platform.
3. Input the `HERMES_BOT_TOKEN`.
4. Input your **Discord User ID**.
5. Start the gateway service:
   ```bash
   hermes gateway start
   ```

### 3. Bot-to-Bot Mention Protocol (`n8n-bot` -> `hermes-bot`)
In the Development Agent and Analytics Agent workflows, `n8n-bot` triggers `hermes-bot` by posting a message formatted with `hermes-bot`'s Discord Bot User ID mention tag:

```text
=<@HERMES_BOT_USER_ID> the generated GitHub Pages website specification is attached...
```

- When `n8n-bot` posts this message with the attached specification `.md` file, `hermes-bot` detects the `@mention`, downloads the attached file, and initiates the automated GitHub repository build.

---

## Security & Operational Guidelines

> **Important: Bot Role Separation**
> Operating two separate bots (`n8n-bot` and `hermes-bot`) prevents event loop deadlocks (e.g., Hermes responding to automated n8n notifications) and ensures clear audit trails for server actions.

> **Note: Channel Mention Behavior**
> - `hermes-bot`: In server channels, Hermes requires an explicit `@hermes-bot` mention to trigger execution. In direct messages (DMs), it responds automatically.
> - `n8n-bot`: Listens to target channel webhooks or specific command prefixes configured in n8n trigger nodes.

---

## Related Documents
- [Overview Page](file:///Users/seerneil/Documents/codespaces/ailab/synergy/overview.md)
- Previous Step: [Google Account & Cloud Console Setup](ref/google-account.md)
- Next Step: [GitHub PAT & GitHub Pages Guide](ref/github.md)
