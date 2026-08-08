# GitHub Personal Access Token & GitHub Pages Setup Guide

> **Setup Navigation**: [Step 3: Discord Dual-Bot Setup](discord.md) | **Step 4: GitHub PAT & GitHub Pages Setup** | [Step 5: Hermes Agent Setup](hermes.md)

This guide details how to generate a **GitHub Personal Access Token (PAT Classic)** with appropriate scope permissions and how to set up **GitHub Pages** for the **Synergy Marketing Ecosystem** — hosting automated web outputs across your repositories.

> 💡 **Non-Technical Note:**  

> **What is a GitHub PAT & GitHub Pages?** A **Personal Access Token (PAT)** is a pass key that lets automated agent CLI tools commit code to your account. **GitHub Pages** is a free hosting feature that turns any website files stored inside your repository's `/docs` folder into a live, public website URL. Need more definitions? See **[Synergy Glossary](glossary.md)**.

---

## Visual PAT Setup Guide

![GitHub Personal Access Token Classic Setup](/Users/seerneil/.gemini/antigravity-ide/brain/b7272dad-971c-4782-acc8-c5486b78e963/github_pat_setup_guide_1786177306358.png)

---

## Tool Overview & Ecosystem Purpose

| Property | Details |
| :--- | :--- |
| **Tool Name** | **GitHub & GitHub Pages** |
| **Tool Classification** | Web-based developer platform for version control (Git), source code hosting, and static web page hosting. |
| **License Type** | Proprietary Cloud Platform / Freemium (GitHub CLI `gh` is Open Source under MIT License) |
| **Purpose in Ecosystem** | Acts as the primary source code repository manager, handling Personal Access Token (PAT) authentication for automated agents, receiving code commits from Hermes Agent, and serving live websites via GitHub Pages (`/docs`). |

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

---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| `git push` or `gh auth login` returns `403 Forbidden` / `Bad Credentials` | PAT lacks the mandatory `repo` scope or the token has expired. | Go to GitHub **Settings > Developer settings > Personal access tokens (classic)**, verify expiration status, and generate a new token with `repo` scope checked. |
| Published GitHub Pages URL displays `404 Not Found` | Repository lacks an `index.html` file inside the `/docs` folder on `main` branch, or deployment is in progress. | Create `docs/index.html` on `main` branch. Go to **Settings > Pages**, verify source is set to `Deploy from a branch` -> `main` -> `/docs`, and check **Actions** build progress. |
| Git commit fails with `fatal: unable to auto-detect email address` | Git user identity (`user.name` and `user.email`) has not been configured on host/VM. | Run `git config --global user.name "Hermes Agent Bot"` and `git config --global user.email "hermes-bot@users.noreply.github.com"` in terminal. |
| `gh auth login` hangs waiting for browser or token input in automation scripts | Script invokes interactive mode instead of non-interactive token pipe. | Pipe the PAT environment variable directly into the CLI: `echo "$GITHUB_TOKEN" | gh auth login --with-token`. |
| Git push rejected with `[rejected - non-fast-forward]` error | Remote repository branch contains remote commits not present in local working copy. | Run `git pull --rebase origin main` to integrate remote changes before executing `git push origin main`. |
| `ssh: connect to host github.com port 22: Connection refused` | Firewall or network proxy blocks outgoing SSH connections on port 22. | Switch git remote URL from SSH to HTTPS: `git remote set-url origin https://github.com/<username>/<repo>.git` or use `gh auth login` with HTTPS protocol. |
| GitHub Pages build fails with `Jekyll build error` | Unescaped liquid tags or syntax errors in static Markdown/HTML files inside `/docs`. | Add `.nojekyll` file to the root of your repository or `/docs` folder to bypass default Jekyll processing. |

---

## Related Documents
- [Overview Page](../overview.md)
- Previous Step: [Discord Developer & Bot Setup](discord.md)
- Next Step: [Hermes Agent Installation & Integration](hermes.md)



