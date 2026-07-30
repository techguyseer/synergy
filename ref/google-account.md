# Google Account & Cloud Console OAuth Setup Guide

This guide details how to configure a Google Cloud Console project for the **Synergy Marketing Ecosystem**, enabling required APIs for **Gmail**, **Google Drive**, and **Google Docs**, and creating **OAuth 2.0 Credentials (Client ID & Client Secret)** for integration into **n8n** nodes.

---

## Overview

n8n Google nodes (Gmail, Google Drive, Google Docs) use OAuth 2.0 authentication to securely perform operations on behalf of your Google account. This requires establishing an application entry in Google Cloud Console.

---

## Step 1: Create a Google Cloud Project

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Log in with the Google Account that holds or will access the targeted Mail, Drive, and Docs resources.
3. In the top navigation bar, click the project selector dropdown and click **New Project**.
4. Enter a descriptive **Project Name** (e.g., `n8n-multi-agent-automation`).
5. Select an Organization or Location if applicable, then click **Create**.
6. Ensure your newly created project is selected in the top dropdown bar before proceeding.

---

## Step 2: Enable Required Google Workspace APIs

To allow n8n agents to interact with Gmail, Google Drive, and Google Docs, you must explicitly enable each API.

1. Navigate to **APIs & Services > Library** from the left navigation menu.
2. Search for and enable the following three APIs one by one:
   - **Gmail API** -> Click **Enable**.
   - **Google Drive API** -> Click **Enable**.
   - **Google Docs API** -> Click **Enable**.

---

## Step 3: Configure the OAuth Consent Screen

Before creating OAuth credentials, Google requires an OAuth Consent Screen setup.

1. In the left sidebar, navigate to **APIs & Services > OAuth consent screen**.
2. Select **User Type**:
   - **Internal**: Available only if you are using a Google Workspace organization account. (Simplest option; no verification required).
   - **External**: Required if using a standard `@gmail.com` personal account.
3. Click **Create**.
4. Fill in the **App Information**:
   - **App name**: `n8n Integration` (or your preferred name).
   - **User support email**: Select your email address.
   - **Developer contact information**: Enter your email address.
5. Click **Save and Continue**.
6. **Scopes**:
   - Click **Add or Remove Scopes**.
   - Search for and select the scopes matching your workflow requirements:
     - `https://mail.google.com/` (Gmail read/send/manage)
     - `https://www.googleapis.com/auth/drive` (Drive file access)
     - `https://www.googleapis.com/auth/documents` (Docs read/write)
   - Click **Update** and then **Save and Continue**.
7. **Test Users** (Critical for External / Testing Status):
   - Click **+ Add Users**.
   - Enter your Google email address and any additional account emails that will authenticate with n8n.
   - Click **Add**, then click **Save and Continue**.

> **Warning: Testing Status Expiration Limit**
> If your OAuth Consent Screen remains in **Testing** status with an **External** user type, Google automatically expires refresh tokens after **7 days**. You will need to re-authenticate credentials in n8n weekly unless you move the app to **Production** status or use an **Internal** Workspace account.

---

## Step 4: Create OAuth 2.0 Credentials (Client ID & Client Secret)

1. In the left sidebar, click **APIs & Services > Credentials**.
2. Click **+ Create Credentials** at the top and select **OAuth client ID**.
3. Set **Application type** to **Web application**.
4. Set **Name** to `n8n OAuth Client`.
5. Scroll down to **Authorized redirect URIs**:
   - Click **+ Add URI**.
   - Enter your exact n8n OAuth callback URL:
     - For **n8n Cloud**: `https://oauth.n8n.cloud/rest/oauth2-credential/callback`
     - For **Self-Hosted n8n**: `https://<YOUR_N8N_DOMAIN>/rest/oauth2-credential/callback`
6. Click **Create**.
7. A modal will display your **Client ID** and **Client Secret**.
8. Copy both values immediately and store them securely in a password manager.

---

## Step 5: Integrating Credentials into n8n Google Nodes

Now connect your credentials to n8n for the three distinct Google services used in the workflows:

1. Open your **n8n Web Interface** and navigate to **Credentials > Add Credential**.
2. Create and authenticate credentials for each of the following credential types:
   - **Gmail OAuth2 API** (`gmailOAuth2`): Used by the Orchestrator to compose and send campaign delivery emails. Name it `Gmail account`.
   - **Google Docs OAuth2 API** (`googleDocsOAuth2Api`): Used by the Marketing Document Agent to create documents and perform batch updates. Name it `Google Docs account`.
   - **Google Drive OAuth2 API** (`googleDriveOAuth2Api`): Used by the Marketing Document Agent to share documents publicly ("anyone as reader"). Name it `Google Drive account`.
3. For each credential type, paste your **Client ID** and **Client Secret**, click **Sign in with Google**, grant requested permissions, and click **Save**.

---

## Step 6: Creating a Custom Google Drive Campaign Folder

In the Marketing Document Agent workflow, generated marketing plans are stored inside a dedicated Google Drive folder:

1. Open [Google Drive](https://drive.google.com).
2. Click **+ New > New folder** and name it `Marketing Campaign Briefs`.
3. Open the folder and examine the browser URL:
   `https://drive.google.com/drive/folders/<YOUR_FOLDER_ID>`
4. Copy the long alphanumerical `<YOUR_FOLDER_ID>` string from the URL bar.
5. In n8n, open the `Create Marketing Plan Doc` node inside the Marketing Document Agent workflow, and paste your folder ID into the **Folder ID** field.

---

## Security & Best Practices

> **Important: Secret Key Confidentiality**
> - Your **Client Secret** grants full API access scoped to your consent screen configuration. Never share or commit Client Secrets to public repositories or unencrypted scripts.

> **Note: Publishing App vs. Verification**
> - If you publish an External app to production to eliminate the 7-day token limit, Google may require app verification for sensitive scopes (`mail.google.com` or `drive`). For personal automation workflows, maintaining the app in **Testing** mode with explicit **Test Users** is standard practice.

---

## Related Documents
- [Overview Page](file:///Users/seerneil/Documents/codespaces/ailab/synergy/overview.md)
- Previous Step: [Ollama Setup Guide](ref/ollama.md)
- Next Step: [Discord Developer & Bot Setup](ref/discord.md)
