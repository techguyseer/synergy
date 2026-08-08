# Discord Developer & Dual-Bot Integration Guide

> **Setup Navigation**: [Step 2: Google Credentials](google-account.md) | **Step 3: Discord Developer & Dual-Bot Setup** | [Step 4: GitHub & GitHub Pages Setup](github.md)

This document outlines the step-by-step process for creating **two separate Discord Developer Applications and Bots** for the **Synergy Marketing Ecosystem** — one named **`n8n-bot`** for n8n Workflows and one named **`hermes-bot`** for Hermes Agent.

> 💡 **Non-Technical Note:**  

> **What are Bot Tokens & Intents?** A **Bot Token** is like a secret login key for your bot assistant. **Privileged Gateway Intents** (Message Content Intent) are master switches that grant the bot permission to read incoming channel text so it can respond to prompts. Need more definitions? See **[Synergy Glossary](glossary.md)**.

---

## Tool Overview & Ecosystem Purpose

| Property | Details |
| :--- | :--- |
| **Tool Name** | **Discord** |
| **Tool Classification** | Instant messaging, Voice-over-IP (VoIP), and digital team collaboration platform. |
| **License Type** | Proprietary / Freemium |
| **Purpose in Ecosystem** | Serves as the central communication and event gateway for the Synergy Marketing Ecosystem, hosting **`n8n-bot`** (for automated channel alerts & specification dispatches) and **`hermes-bot`** (for interactive AI execution & prompt management). |

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

### 1. Obtain Your Discord User ID (For `DISCORD_ALLOWED_USERS`)
1. Open Discord, go to **User Settings (gear icon) > Advanced**.
2. Toggle **Developer Mode** to **ON**.
3. Right-click your profile avatar/name in Discord and click **Copy User ID**.
4. Save this numerical ID (e.g., `123456789012345678`) to populate `DISCORD_ALLOWED_USERS` (or `HERMES_ALLOWED_USERS`).

### 2. Obtain Your Target Channel ID & Guild ID (For `DISCORD_ALLOWED_CHANNELS` & n8n)
1. In your Discord server, right-click the text channel designated for agent communications (e.g., `#agent-lab`).
2. Click **Copy Channel ID** and save the ID (e.g., `112233445566778899`) to populate `DISCORD_ALLOWED_CHANNELS`.
3. Right-click your Server icon in the server list and click **Copy Server ID** (Guild ID).
4. In n8n, update the `Send a message to Hermes` node in the Development Agent and Analytics Agent workflows with your Server ID and Channel ID.

> 💡 **Fail-Closed Security Model & Debugging Override Notice:**
> `DISCORD_ALLOWED_USERS` is a **required configuration field** due to Hermes Agent's fail-closed security model. If omitted, the bot ignores all incoming messages.
> For **debugging or initial testing**, set **`DISCORD_ALLOWED_USERS=true`** or **`DISCORD_ALLOW_ALL_USERS=true`** so Hermes Agent responds to any Discord user without filtering.

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
3. Input the `DISCORD_BOT_TOKEN`.
4. Set **Allowed Users** (`DISCORD_ALLOWED_USERS`): Enter numerical User IDs *(or enter `true` / set `DISCORD_ALLOW_ALL_USERS=true` for debugging)*.
5. Set **Allowed Channels** (`DISCORD_ALLOWED_CHANNELS`): Enter numerical Channel IDs *(or leave blank for all channels)*.
6. Start the gateway service:
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

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| `hermes-bot` ignores `@mention` tags in Discord server channels | **Message Content Intent** is disabled in Discord Developer Portal or channel read permissions are missing. | Go to [Discord Developer Portal](https://discord.com/developers/applications) > `hermes-bot` > **Bot**, enable **Message Content Intent**, and grant `View Channels` and `Read Message History` permissions in channel settings. |
| `hermes-bot` does not trigger when `n8n-bot` posts a message | Discord suppresses bot-to-bot notifications when using display names instead of raw user ID tags. | Format the mention using raw Discord User ID syntax `<@HERMES_BOT_USER_ID>` (e.g., `<@123456789012345678>`) inside the n8n message payload. |
| `n8n-bot` fails with `403 Forbidden` / `Missing Permissions` when posting | Bot role lacks `Send Messages`, `Embed Links`, or `Attach Files` permissions in the target channel. | Re-generate the bot OAuth2 invite URL with necessary scopes, or edit Channel Settings > Permissions > Add Bot Role > Allow `Send Messages` and `Attach Files`. |
| `hermes-bot` appears **Offline** in Discord server member list | Hermes Agent gateway daemon is not running on the host machine. | Open terminal on your host or VM, run `hermes gateway status`, and start the gateway service using `hermes gateway start`. |
| `n8n-bot` fails to attach specification `.md` files to Discord channel | Bot lacks `Attach Files` scope or file exceeds Discord's free tier attachment size limit (25 MB). | Verify `Attach Files` permission is enabled in OAuth2 settings and ensure attached markdown spec files are under 25 MB. |
| Discord returns `429 Too Many Requests` (Rate Limited) | n8n workflow sends messages in a rapid loop without delay parameters. | Add a **Wait** node (1–2 seconds) between iterative message dispatch loops in n8n to comply with Discord API rate limits. |
| Cannot locate `hermes-bot` User ID (Application ID copied by mistake) | Discord Developer Mode is turned off, or Application Client ID was copied instead of Bot User ID. | Enable **Developer Mode** in Discord Settings > User Settings > Advanced. Right-click `hermes-bot` in the server member list and click **Copy User ID**. |

---

## Related Documents
- [Overview Page](../overview.md)
- Previous Step: [Google Account & Cloud Console Setup](google-account.md)
- Next Step: [GitHub PAT & GitHub Pages Guide](github.md)



