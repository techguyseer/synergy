# GitHub Personal Access Token & GitHub Pages Setup Guide

This guide details how to generate a **GitHub Personal Access Token (PAT Classic)** with appropriate scope permissions and how to set up **GitHub Pages** for the **Synergy Marketing Ecosystem** — hosting automated web outputs across your repositories.

---

## Overview

Your automated agents require authentication to interact with GitHub repositories—such as committing updated code, cloning repos, or managing documentation. GitHub Pages allows agents to publish static web pages directly from repository branches.

---

## Part 1: Generating a Personal Access Token (PAT Classic)

GitHub PATs act as secure authentication tokens for scripts, the GitHub CLI (`gh`), n8n GitHub nodes, and automated workflow agents.

### 1. Navigation & Creation
1. Log in to your account on [GitHub.com](https://github.com).
2. Click your profile avatar in the upper right corner and select **Settings**.
3. In the left sidebar, scroll down to the bottom and click **Developer settings**.
4. Navigate to **Personal access tokens > Tokens (classic)**.
5. Click **Generate new token** and select **Generate new token (classic)**.

---

### 2. Token Configuration & Scope Selection
1. **Note**: Give your token a clear identifier (e.g., `Hermes-n8n-Automation-Token`).
2. **Expiration**: Select an expiration period aligned with your security policy (e.g., 90 days, or No expiration for dedicated service accounts).
3. **Select Scopes**:
   - `repo` (Full control of private and public repositories) -> **Mandatory**: Grants access to repository code, commit status, webhooks, and GitHub Pages deployments.
   - `workflow` (Update GitHub Action workflows) -> **Recommended**: Required if your agents modify `.github/workflows/`.
   - `read:user` & `user:email` -> **Recommended**: Allows agents to read user profile info and configure git user signatures.
4. Click **Generate token** at the bottom of the page.

---

### 3. Securing Your Token
1. **Copy the token value immediately**. GitHub will not display this token again once you navigate away from the page.
2. Store the token in a secure password manager or credential vault.

> **Warning: Token Security Notice**
> Never hardcode your GitHub Personal Access Token inside committed code files or public repositories. Exposing a PAT with `repo` scopes allows full unauthorized access to your source code and repositories.

---

## Part 2: Creating Custom Repositories & Enabling GitHub Pages

As part of your laboratory exercises, you will create **two personal GitHub repositories**—one for the marketing website prototype and one for the analytics dashboard prototype.

### 1. Creating Your Custom Repositories
Create two public repositories on GitHub:

1. **Website Repository**: Go to [github.com/new](https://github.com/new), set repository name (e.g., `website-prototype`), set visibility to **Public**, check **Add a README file**, and click **Create repository**.
2. **Analytics Repository**: Go to [github.com/new](https://github.com/new), set repository name (e.g., `analytics-prototype`), set visibility to **Public**, check **Add a README file**, and click **Create repository**.

---

### 2. Enabling GitHub Pages Deployment (`/docs` Folder)
Perform these steps for **both** created repositories:

1. Open your repository on GitHub.
2. Click the **Settings** tab in the repository navigation bar.
3. In the left sidebar under **Code and automation**, click **Pages**.
4. Under **Build and deployment**:
   - **Source**: Select **Deploy from a branch**.
   - **Branch**: Select `main`.
   - **Folder**: Select `/docs` (Hermes Agent will generate and commit site assets into the `docs/` folder).
5. Click **Save**.

---

### 3. Verifying Live Site URLs & Updating n8n Prompts
1. Refresh **Settings > Pages** after 1–2 minutes to verify your published live URL format:
   `https://<your-username>.github.io/<your-repository-name>/`
2. **Updating n8n Workflows**:
   - Open the Development Agent workflow in n8n, locate the Discord node prompt, and replace the sample clone URL with your custom HTTPS repository URL: `https://github.com/<your-username>/website-prototype.git`.
   - Open the Analytics Agent workflow in n8n, locate the Discord node prompt, and replace the sample clone URL with your custom HTTPS repository URL: `https://github.com/<your-username>/analytics-prototype.git`.

---

## Part 3: Using PAT Credentials in n8n & CLI Tools

### 1. n8n GitHub Node
- In n8n, navigate to **Credentials > Add Credential** and select `GitHub API`.
- Select **Personal Access Token** as the authentication method.
- Paste your generated PAT into the token field and click **Save**.

### 2. Git CLI Configuration
To authenticate git commands in terminal environments:
```bash
# Standard HTTPS authentication format
git clone https://<your-username>:<YOUR_PAT_TOKEN>@github.com/<your-username>/<repo-name>.git
```

---

## Security & Operational Guidelines

> **Important: PAT Scope Granularity**
> If you only require an agent to modify a single specific repository, consider creating a **Fine-grained Personal Access Token** under Developer Settings, scoped exclusively to that target repository instead of all account repositories.

---

## Related Documents
- [Overview Page](file:///Users/seerneil/Documents/codespaces/ailab/synergy/overview.md)
- Previous Step: [Discord Developer & Bot Setup](ref/discord.md)
- Next Step: [Hermes Agent Installation & Integration](ref/hermes.md)
