# OpenRouter Setup & Model Routing Guide

This guide details how to create an **OpenRouter** account, generate **API Keys**, navigate **Free and Paid Model options**, and configure **Model Auto-Routing** for cloud LLM inference within the **Synergy Marketing Ecosystem**.

---

## Part 1: OpenRouter Overview & Account Creation

OpenRouter is a unified AI model gateway that provides a standardized API for accessing open-source and commercial LLMs (from Meta, Google, Anthropic, OpenAI, Qwen, DeepSeek, and Mistral).

### 1. Creating an OpenRouter Account
1. Navigate to [openrouter.ai](https://openrouter.ai).
2. Click **Sign In** in the top right corner.
3. Authenticate using your **GitHub account**, **Google account**, or **Email address**.
4. Complete account verification if prompted.

---

## Part 2: Generating & Managing API Keys

1. Navigate to **Account Settings > Keys**: [openrouter.ai/keys](https://openrouter.ai/keys).
2. Click **Create Key**.
3. Enter a descriptive key name (e.g., `synergy-marketing-ecosystem`).
4. Optional: Set a **Credit Limit** to prevent unexpected usage costs.
5. Click **Create**.
6. **Copy the API key immediately** (`sk-or-v1-...`) and store it securely in a password manager.

> **Security Warning**
> Never commit OpenRouter API keys to public repositories or hardcode them in source files. Inject your key via environment variables (`OPENROUTER_API_KEY`) or n8n's secure credential store.

---

## Part 3: Selecting Models (Free vs. Paid Options)

OpenRouter provides access to both zero-cost (free tier) models and pay-per-token commercial models.

### 1. Free Tier Models (`:free` suffix)
OpenRouter offers rate-limited, zero-cost access to selected open-weight models. Free models feature the `:free` suffix in their model identifier:

| Model ID | Context Window | Use Case |
| :--- | :--- | :--- |
| `meta-llama/llama-3.3-70b-instruct:free` | 128k tokens | High-reasoning marketing planning & document drafting |
| `google/gemini-2.0-flash-exp:free` | 1M tokens | Fast spec generation & high-speed JSON routing |
| `qwen/qwen-2.5-coder-32b-instruct:free` | 32k tokens | HTML/CSS/JS web code generation for Hermes Agent |
| `deepseek/deepseek-r1:free` | 64k tokens | Complex reasoning & analytical projection calculations |

Browse all currently available free models at: [openrouter.ai/models?max_price=0](https://openrouter.ai/models?max_price=0).

### 2. Paid Commercial Models
For production-grade availability and higher rate limits, you can add credits to your account balance under **Account > Credits** ([openrouter.ai/credits](https://openrouter.ai/credits)).

| Model ID | Provider | Ideal Ecosystem Role |
| :--- | :--- | :--- |
| `anthropic/claude-3.5-sonnet` | Anthropic | Complex website & dashboard code generation |
| `openai/gpt-4o` | OpenAI | Multimodal strategy and JSON schema compliance |
| `google/gemini-flash-1.5` | Google | Low-cost high-throughput orchestrator routing |

---

## Part 4: Model Routing & Auto-Routing

OpenRouter includes native features for dynamic model selection, fallback chains, and automatic performance routing.

### 1. OpenRouter Auto-Routing (`openrouter/auto`)
When using `openrouter/auto` as your model string, OpenRouter evaluates incoming prompts and automatically routes requests to the optimal available model based on prompt length, model availability, latency, and cost efficiency.

```json
{
  "model": "openrouter/auto",
  "messages": [
    { "role": "user", "content": "Generate a website specification..." }
  ]
}
```

### 2. Explicit Fallback Chains
You can supply an ordered list of candidate models in your API request. If the primary model encounters rate limits or downtime, OpenRouter automatically falls back to subsequent models in your list:

```json
{
  "models": [
    "meta-llama/llama-3.3-70b-instruct:free",
    "google/gemini-2.0-flash-exp:free",
    "qwen/qwen-2.5-coder-32b-instruct:free"
  ],
  "messages": [
    { "role": "user", "content": "Synthesize marketing campaign data..." }
  ]
}
```

---

## Part 5: Integration Points & Fallback Methods in n8n

### 1. Specifying Model Fallbacks in n8n

There are three ways to configure LLM fallbacks inside n8n:

#### Method A: Server-Side Comma-Separated Model String (Recommended)
Inside n8n OpenAI / LLM Model nodes, enter a comma-separated list of models in the **Model** input field:
```text
meta-llama/llama-3.3-70b-instruct:free,google/gemini-2.0-flash-exp:free,qwen/qwen-2.5-coder-32b-instruct:free
```
OpenRouter automatically attempts the models in order server-side before returning the payload to n8n.

#### Method B: Visual n8n Node Error Branching (`On Error`)
1. Open your primary LLM node in n8n.
2. Go to **Settings > On Error** and select **`Continue (using error output)`**.
3. Connect the secondary red error output to a fallback LLM node (e.g., local Ollama node or secondary OpenRouter model).

#### Method C: Auto-Routing (`openrouter/auto`)
Set the **Model** field in n8n to `openrouter/auto`. OpenRouter handles provider selection and automatic failover dynamically.

### 2. n8n OpenAI Credentials Setup
OpenRouter provides an OpenAI-compatible Base URL (`https://openrouter.ai/api/v1`).

1. In n8n, go to **Credentials > Add Credential > OpenAI API**.
2. Set **API Key**: `sk-or-v1-...`
3. For custom HTTP Request nodes:
   - **URL**: `https://openrouter.ai/api/v1/chat/completions`
   - **Header**: `Authorization: Bearer OPENROUTER_API_KEY`
   - **Header**: `HTTP-Referer: https://github.com/techguyseer/synergy`

### 3. Hermes Agent Integration
OpenRouter can serve as the primary cloud LLM provider for Hermes Agent on hosts without local GPU acceleration:

1. Set the environment variable in your Hermes shell or `~/.config/hermes/config.yaml`:
   ```bash
   export OPENROUTER_API_KEY="sk-or-v1-your-key-here"
   ```
2. Configure Hermes model provider:
   ```bash
   hermes config set model_provider openrouter
---

## Common Troubleshooting Guide

| Symptom / Error Message | Probable Cause | Recommended Solution |
| :--- | :--- | :--- |
| OpenRouter returns `401 Unauthorized` or `402 Payment Required` | Invalid API key (`401`) or invoking a paid commercial model with `$0.00` credit balance (`402`). | Verify key at [openrouter.ai/keys](https://openrouter.ai/keys). Ensure model name includes `:free` suffix (e.g. `meta-llama/llama-3.3-70b-instruct:free`) if balance is `$0.00`. |
| `429 Too Many Requests` rate limit on free tier models | Global per-minute free request caps reached during peak usage hours across users. | Set `model` to `openrouter/auto` or provide a comma-separated fallback candidate string (`meta-llama...:free,google/gemini...:free`). |
| Custom n8n HTTP Request node rejected by OpenRouter | Missing mandatory site attribution headers required by OpenRouter API. | Add headers: `HTTP-Referer: https://github.com/techguyseer/synergy` and `X-Title: Synergy Marketing Ecosystem`. |
| OpenRouter API call times out after 60+ seconds | Upstream model provider latency or server overload on requested free model. | Define explicit provider fallbacks or switch primary model to a faster endpoint like `google/gemini-2.0-flash-exp:free`. |
| LLM returns truncated / incomplete JSON strategy payload | Model max output token parameter limit reached during response generation. | Increase `max_tokens` parameter (e.g. 4096) and enforce JSON mode formatting instructions in the system prompt. |
| `openrouter/auto` selects unexpected or low-quality model | System prompt lacks clear complexity indicators for router model matching. | Pass explicit candidate model array `["meta-llama/llama-3.3-70b-instruct:free", "google/gemini-2.0-flash-exp:free"]` instead of auto. |
| Account balance drops unexpectedly on paid key | Uncapped API key used in recursive loop or auto-retrying n8n node. | Navigate to [openrouter.ai/keys](https://openrouter.ai/keys) and set a strict spending cap limit on your API key. |

---

## Related Documents
- [Overview Page](../overview.md)
- [Ollama Setup Guide](ollama.md)
- Next Step: [Google Account & Cloud Console Setup](google-account.md)




