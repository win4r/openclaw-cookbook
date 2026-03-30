# 03 — Model Providers

**Configure LLM backends with fallbacks, per-agent routing, and cost optimization.**

Time: ~20 minutes

---

## Overview

OpenClaw does not call one model. It manages a **provider pool** -- a set of configured LLM backends that your agents draw from. Each agent can be assigned a different model, and each model can have fallback chains so your gateway stays up even when a provider goes down.

```
┌─────────────────────────────────────────────────────────┐
│                   OpenClaw Gateway                      │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │  Agent A  │  │  Agent B  │  │  Agent C  │             │
│  │ claude-   │  │ gpt-5.4  │  │ kimi-k2.5 │             │
│  │ sonnet-4-6│  │          │  │           │             │
│  └─────┬─────┘  └─────┬────┘  └─────┬─────┘             │
│        │               │              │                  │
│  ┌─────▼───────────────▼──────────────▼─────┐           │
│  │          Model Provider Pool              │           │
│  │                                           │           │
│  │  ┌───────────┐  ┌────────┐  ┌──────────┐│           │
│  │  │ Anthropic  │  │ OpenAI │  │ MiniMax  ││           │
│  │  │ (Claude)   │  │ (GPT)  │  │          ││           │
│  │  └───────────┘  └────────┘  └──────────┘│           │
│  │  ┌───────────┐  ┌────────┐  ┌──────────┐│           │
│  │  │   Kimi    │  │DeepSeek│  │  Ollama  ││           │
│  │  │(Moonshot) │  │        │  │ (local)  ││           │
│  │  └───────────┘  └────────┘  └──────────┘│           │
│  └───────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────┘
```

Three concepts:

1. **Provider** -- an API endpoint with credentials (Anthropic, OpenAI, etc.)
2. **Model assignment** -- which model each agent uses as its primary
3. **Fallback chain** -- ordered list of models to try when the primary fails

All configuration lives in `openclaw.json` at your workspace root. API keys live in `~/.openclaw/.env`.

---

## Provider Setup

### Anthropic (Claude)

The default and most common provider.

**1. Set your API key:**

```bash
echo 'ANTHROPIC_API_KEY=sk-ant-api03-...' >> ~/.openclaw/.env
```

**2. Configure the provider in `openclaw.json`:**

```json
{
  "models": {
    "providers": {
      "anthropic": {
        "api": "anthropic-messages",
        "apiKey": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

**Available models:**

| Model | Best For | Context Window |
|-------|----------|----------------|
| `claude-opus-4-6` | Complex reasoning, coding, analysis | 1M tokens |
| `claude-sonnet-4-6` | Balanced speed and quality | 200K tokens |
| `claude-haiku-3-5` | Fast responses, simple tasks | 200K tokens |

**Set as default:**

```json
{
  "agents": {
    "list": [
      { "id": "default", "name": "Default", "model": { "primary": "anthropic/claude-sonnet-4-6" } }
    ]
  }
}
```

---

### OpenAI (GPT)

**1. Set your API key:**

```bash
echo 'OPENAI_API_KEY=sk-proj-...' >> ~/.openclaw/.env
```

**2. Configure the provider:**

```json
{
  "models": {
    "providers": {
      "openai": {
        "api": "openai-chat",
        "apiKey": "${OPENAI_API_KEY}"
      }
    }
  }
}
```

**Available models:**

| Model | Best For | Context Window |
|-------|----------|----------------|
| `gpt-5.4` | Frontier reasoning and coding | 256K tokens |
| `gpt-5.2` | Strong general purpose | 256K tokens |
| `gpt-4.1` | Reliable, well-tested | 128K tokens |
| `gpt-4.1-mini` | Fast and cheap | 128K tokens |
| `gpt-4.1-nano` | Cheapest, simple tasks | 128K tokens |

---

### MiniMax

MiniMax uses an **Anthropic-compatible API** at `api.minimaxi.com`. You configure it as a custom Anthropic endpoint.

**1. Set your API key:**

```bash
echo 'MINIMAX_API_KEY=eyJ...' >> ~/.openclaw/.env
```

**2. Configure the provider:**

```json
{
  "models": {
    "providers": {
      "minimax": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "api": "anthropic-messages",
        "authHeader": true,
        "models": [
          { "id": "MiniMax-M2.7", "name": "MiniMax M2.7", "contextWindow": 204800 }
        ]
      }
    }
  }
}
```

**Available models:**

| Model | Best For | Context Window |
|-------|----------|----------------|
| `MiniMax-M2.7` | Long-form content, creative writing | 200K tokens |

Because MiniMax uses the Anthropic-compatible API format, you reference its models with the same syntax as any Anthropic model. Set the model name in the agent's model assignment:

```json
{
  "agents": {
    "list": [
      { "id": "default", "name": "Default", "model": { "primary": "minimax/MiniMax-M2.7" } }
    ]
  }
}
```

---

### Kimi / Moonshot

**1. Set your API key:**

```bash
echo 'MOONSHOT_API_KEY=sk-...' >> ~/.openclaw/.env
```

**2. Configure the provider:**

```json
{
  "models": {
    "providers": {
      "moonshot": {
        "api": "openai-chat",
        "apiKey": "${MOONSHOT_API_KEY}",
        "baseUrl": "https://api.moonshot.cn/v1"
      }
    }
  }
}
```

**Available models:**

| Model | Best For | Context Window |
|-------|----------|----------------|
| `kimi-k2.5` / `k2p5` | Chinese + English bilingual tasks | 128K tokens |
| `moonshot-v1-128k` | Long-document analysis | 128K tokens |
| `moonshot-v1-32k` | General purpose | 32K tokens |
| `moonshot-v1-8k` | Fast, simple tasks | 8K tokens |

---

### DeepSeek

DeepSeek provides an OpenAI-compatible API.

**1. Set your API key:**

```bash
echo 'DEEPSEEK_API_KEY=sk-...' >> ~/.openclaw/.env
```

**2. Configure the provider:**

```json
{
  "models": {
    "providers": {
      "deepseek": {
        "api": "openai-chat",
        "apiKey": "${DEEPSEEK_API_KEY}",
        "baseUrl": "https://api.deepseek.com/v1"
      }
    }
  }
}
```

**Available models:**

| Model | Best For | Context Window |
|-------|----------|----------------|
| `deepseek-chat` | General conversation | 64K tokens |
| `deepseek-reasoner` | Complex reasoning (R1) | 64K tokens |

---

### Ollama (Local)

Run models locally. No API key, no data leaves your machine. Best for privacy-sensitive deployments.

**1. Install and start Ollama:**

```bash
# macOS
brew install ollama
ollama serve

# Pull a model
ollama pull llama3.1:70b
ollama pull qwen2.5:32b
```

**2. Configure as an OpenAI-compatible provider (no API key needed):**

```json
{
  "models": {
    "providers": {
      "ollama": {
        "api": "openai-chat",
        "apiKey": "ollama",
        "baseUrl": "http://localhost:11434/v1"
      }
    }
  }
}
```

> **Note:** The `apiKey` field is required by the config schema but Ollama ignores it. Set it to any non-empty string.

**Recommended local models:**

| Model | Size | Best For |
|-------|------|----------|
| `llama3.1:70b` | ~40 GB | Strong general purpose |
| `llama3.1:8b` | ~5 GB | Fast, lightweight |
| `qwen2.5:32b` | ~18 GB | Good bilingual (EN/ZH) |
| `codestral:22b` | ~13 GB | Code generation |
| `deepseek-r1:32b` | ~18 GB | Reasoning tasks |

**Hardware requirements:** 70B models need 48+ GB RAM (or Apple Silicon with 48+ GB unified memory). 8B models run comfortably on 16 GB.

---

### Any OpenAI-Compatible API

Any provider that exposes an OpenAI-compatible `/v1/chat/completions` endpoint works the same way. Set `api` to `"openai-chat"`, provide the `baseUrl`, and add a key if required.

```json
{
  "models": {
    "providers": {
      "my-provider": {
        "api": "openai-chat",
        "apiKey": "${MY_PROVIDER_KEY}",
        "baseUrl": "https://my-llm-proxy.internal.corp/v1"
      }
    }
  }
}
```

This covers services like Together AI, Fireworks, Groq, Azure OpenAI, and any self-hosted vLLM or text-generation-inference endpoint.

---

## Fallback Chains

When a model is unavailable (rate limit, outage, timeout), OpenClaw tries the next model in the fallback chain. This keeps your agent responsive even during provider issues.

**How it works:**

```
Request → primary model
           ↓ (fails)
         fallback[0]
           ↓ (fails)
         fallback[1]
           ↓ (fails)
         error returned to user
```

**Configure a fallback chain:**

```json
{
  "agents": {
    "list": [
      {
        "id": "default", "name": "Default",
        "model": {
          "primary": "anthropic/claude-sonnet-4-6",
          "fallbacks": ["openai/gpt-4.1", "deepseek/deepseek-chat"]
        }
      }
    ]
  }
}
```

In this example, if Claude is rate-limited, OpenClaw tries GPT-4.1. If that also fails, it tries DeepSeek. If all three fail, the user gets an error.

**Per-agent fallback chains:**

```json
{
  "agents": {
    "list": [
      {
        "id": "support", "name": "Support",
        "model": {
          "primary": "anthropic/claude-haiku-3-5",
          "fallbacks": ["openai/gpt-4.1-mini"]
        }
      },
      {
        "id": "coding", "name": "Coding",
        "model": {
          "primary": "anthropic/claude-opus-4-6",
          "fallbacks": ["openai/gpt-5.4", "deepseek/deepseek-reasoner"]
        }
      }
    ]
  }
}
```

**Tips for fallback chains:**

- Put cheaper/faster models later in the chain -- they serve as safety nets, not primary choices
- Mix providers (Anthropic + OpenAI + DeepSeek) so a single provider outage does not take you down
- Test your fallbacks by temporarily setting an invalid key for the primary provider

---

## Per-Agent Model Assignment

Different agents have different needs. A support bot answering FAQs does not need the same model as a coding agent writing algorithms.

**The pattern:**

```json
{
  "agents": {
    "list": [
      { "id": "support", "name": "Support", "model": { "primary": "anthropic/claude-haiku-3-5" } },
      { "id": "coding", "name": "Coding", "model": { "primary": "anthropic/claude-opus-4-6" } },
      { "id": "research", "name": "Research", "model": { "primary": "openai/gpt-5.4" } },
      { "id": "local-assistant", "name": "Local Assistant", "model": { "primary": "ollama/llama3.1:70b" } }
    ]
  }
}
```

**How resolution works:**

1. If the agent has an explicit `model.primary` assignment, use it
2. Otherwise, OpenClaw uses the first provider's default model

**Subagent model defaults:**

When an agent spawns subagents (in ClawTeam swarm mode), subagents inherit the parent agent's model by default. Override this to use cheaper models for subagent work:

```json
{
  "agents": {
    "list": [
      {
        "id": "coding", "name": "Coding",
        "model": { "primary": "anthropic/claude-opus-4-6" },
        "subagents": { "model": "anthropic/claude-haiku-3-5" }
      }
    ]
  }
}
```

This means the lead agent thinks with Opus while subagents doing tool calls, file reads, or simple lookups use Haiku -- cutting subagent costs significantly.

---

## Cost Optimization

Model costs vary by orders of magnitude. Smart model assignment is the single biggest lever for controlling your OpenClaw bill.

### Strategy: Tiered Model Assignment

```
Tier 1 (Expensive, high quality)  →  Primary agent, complex tasks
Tier 2 (Moderate)                 →  General agents, most conversations
Tier 3 (Cheap, fast)              →  Subagents, simple tasks, fallbacks
```

### Example: Cost-Optimized Multi-Agent Config

```json
{
  "models": {
    "providers": {
      "anthropic": {
        "api": "anthropic-messages",
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "openai": {
        "api": "openai-chat",
        "apiKey": "${OPENAI_API_KEY}"
      },
      "deepseek": {
        "api": "openai-chat",
        "apiKey": "${DEEPSEEK_API_KEY}",
        "baseUrl": "https://api.deepseek.com/v1"
      }
    }
  },
  "agents": {
    "list": [
      {
        "id": "default", "name": "Default",
        "model": {
          "primary": "anthropic/claude-sonnet-4-6",
          "fallbacks": ["openai/gpt-4.1", "deepseek/deepseek-chat"]
        },
        "subagents": { "model": "anthropic/claude-haiku-3-5" }
      },
      {
        "id": "coding", "name": "Coding",
        "model": {
          "primary": "anthropic/claude-opus-4-6",
          "fallbacks": ["openai/gpt-5.4"]
        },
        "subagents": { "model": "anthropic/claude-sonnet-4-6" }
      },
      {
        "id": "support", "name": "Support",
        "model": {
          "primary": "anthropic/claude-haiku-3-5",
          "fallbacks": ["openai/gpt-4.1-mini", "deepseek/deepseek-chat"]
        }
      }
    ]
  }
}
```

**Cost breakdown (approximate, per 1M tokens input/output):**

| Model | Input | Output | Relative Cost |
|-------|-------|--------|---------------|
| claude-opus-4-6 | $15 | $75 | High |
| gpt-5.4 | $10 | $40 | High |
| claude-sonnet-4-6 | $3 | $15 | Medium |
| gpt-4.1 | $2 | $8 | Medium |
| deepseek-chat | $0.27 | $1.10 | Low |
| claude-haiku-3-5 | $0.80 | $4 | Low |
| gpt-4.1-mini | $0.40 | $1.60 | Low |
| gpt-4.1-nano | $0.10 | $0.40 | Very low |
| Ollama (local) | $0 | $0 | Hardware only |

### Rules of Thumb

- **Support bots** answering FAQs: Haiku or GPT-4.1-mini. Users care about speed, not brilliance.
- **Coding agents**: Opus or GPT-5.4. Correctness saves debugging time.
- **Subagents** doing file reads, tool calls, lookups: Haiku or GPT-4.1-nano. They do not need to be smart.
- **Fallback chains**: End with DeepSeek or a local model. Cheap safety net.
- **Privacy-first deployments**: Ollama with a 70B model. No data leaves the machine.

---

## Provider Comparison Table

| Provider | Speed | Cost Tier | Best For | Context Window | API Compatibility |
|----------|-------|-----------|----------|----------------|-------------------|
| **Anthropic** (Claude) | Medium-Fast | Medium-High | Coding, analysis, instruction following | Up to 1M | Native |
| **OpenAI** (GPT) | Fast | Medium-High | General purpose, broad knowledge | Up to 256K | Native |
| **MiniMax** | Medium | Medium | Long-form content, creative, Chinese | Up to 1M | Anthropic-compatible |
| **Kimi/Moonshot** | Medium | Low-Medium | Bilingual EN/ZH, long documents | Up to 128K | OpenAI-compatible |
| **DeepSeek** | Fast | Low | Budget-conscious, reasoning | Up to 64K | OpenAI-compatible |
| **Ollama** (local) | Varies | Free (hardware) | Privacy, offline, development | Model-dependent | OpenAI-compatible |

**Choosing a provider:**

- Need the best coding/reasoning? Anthropic (Opus) or OpenAI (GPT-5.4).
- Need bilingual Chinese support? Kimi or MiniMax.
- Need the lowest cost? DeepSeek.
- Need zero data egress? Ollama.
- Need maximum uptime? Configure 3+ providers with fallback chains.

---

## Config Reference

Complete multi-provider `openclaw.json` with all features demonstrated:

```json
{
  "models": {
    "providers": {
      "anthropic": {
        "api": "anthropic-messages",
        "apiKey": "${ANTHROPIC_API_KEY}"
      },
      "openai": {
        "api": "openai-chat",
        "apiKey": "${OPENAI_API_KEY}"
      },
      "minimax": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "api": "anthropic-messages",
        "authHeader": true,
        "models": [
          { "id": "MiniMax-M2.7", "name": "MiniMax M2.7", "contextWindow": 204800 }
        ]
      },
      "moonshot": {
        "api": "openai-chat",
        "apiKey": "${MOONSHOT_API_KEY}",
        "baseUrl": "https://api.moonshot.cn/v1"
      },
      "deepseek": {
        "api": "openai-chat",
        "apiKey": "${DEEPSEEK_API_KEY}",
        "baseUrl": "https://api.deepseek.com/v1"
      },
      "ollama": {
        "api": "openai-chat",
        "apiKey": "ollama",
        "baseUrl": "http://localhost:11434/v1"
      }
    }
  },
  "agents": {
    "list": [
      {
        "id": "default", "name": "Default",
        "model": {
          "primary": "anthropic/claude-sonnet-4-6",
          "fallbacks": ["openai/gpt-4.1", "deepseek/deepseek-chat"]
        },
        "subagents": { "model": "anthropic/claude-haiku-3-5" }
      },
      {
        "id": "coding", "name": "Coding",
        "model": {
          "primary": "anthropic/claude-opus-4-6",
          "fallbacks": ["openai/gpt-5.4", "deepseek/deepseek-reasoner"]
        },
        "subagents": { "model": "anthropic/claude-sonnet-4-6" }
      },
      {
        "id": "support", "name": "Support",
        "model": {
          "primary": "anthropic/claude-haiku-3-5",
          "fallbacks": ["openai/gpt-4.1-mini", "moonshot/moonshot-v1-8k"]
        }
      },
      {
        "id": "research", "name": "Research",
        "model": {
          "primary": "openai/gpt-5.4",
          "fallbacks": ["anthropic/claude-sonnet-4-6", "moonshot/kimi-k2.5"]
        },
        "subagents": { "model": "openai/gpt-4.1-mini" }
      },
      {
        "id": "local-private", "name": "Local Private",
        "model": {
          "primary": "ollama/llama3.1:70b",
          "fallbacks": ["ollama/llama3.1:8b"]
        }
      }
    ]
  }
}
```

Corresponding `~/.openclaw/.env`:

```bash
# Model provider API keys
ANTHROPIC_API_KEY=sk-ant-api03-...
OPENAI_API_KEY=sk-proj-...
MINIMAX_API_KEY=eyJ...
MOONSHOT_API_KEY=sk-...
DEEPSEEK_API_KEY=sk-...

# Ollama needs no key -- it runs locally
```

After editing either file, restart the gateway:

```bash
openclaw gateway --restart
```

---

## Troubleshooting

### "Authentication failed" or 401 errors

**Cause:** API key is missing, expired, or incorrectly referenced.

**Fix:**
1. Check that the key exists in `~/.openclaw/.env`:
   ```bash
   cat ~/.openclaw/.env | grep ANTHROPIC
   ```
2. Check that `openclaw.json` references it with the exact variable name:
   ```json
   "apiKey": "${ANTHROPIC_API_KEY}"
   ```
   The `${...}` syntax must match the variable name in `.env` exactly.
3. Make sure the key itself is valid -- test it directly:
   ```bash
   curl https://api.anthropic.com/v1/messages \
     -H "x-api-key: $ANTHROPIC_API_KEY" \
     -H "content-type: application/json" \
     -H "anthropic-version: 2023-06-01" \
     -d '{"model":"claude-haiku-3-5","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
   ```

### "Model not found" errors

**Cause:** The model name does not match what the provider expects.

**Fix:**
- Anthropic models: use exact names like `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-3-5`
- OpenAI models: use `gpt-5.4`, `gpt-5.2`, `gpt-4.1`, `gpt-4.1-mini`, `gpt-4.1-nano`
- Ollama models: use the exact tag you pulled, e.g., `llama3.1:70b` not `llama3.1`
- MiniMax models: use the model name from MiniMax docs, e.g., `MiniMax-M2.7`
- Check for typos in `openclaw.json` -- model names are case-sensitive

### Rate limit errors (429)

**Cause:** You have exceeded the provider's rate limit or token quota.

**Fix:**
- Configure a fallback chain so requests automatically route to another provider
- Check your usage dashboard on the provider's website
- For Anthropic: request a rate limit increase at console.anthropic.com
- For high-traffic bots: use a cheaper primary model and reserve expensive models for complex queries

### Ollama connection refused

**Cause:** Ollama is not running, or it is listening on a different port.

**Fix:**
```bash
# Check if Ollama is running
curl http://localhost:11434/v1/models

# If not, start it
ollama serve

# Verify your model is pulled
ollama list
```

### Fallback chain not triggering

**Cause:** The `fallback` array is misconfigured or the failing provider returns a non-retryable error.

**Fix:**
- Verify the `fallbacks` array is inside the correct agent's `model` block
- Check that fallback model names are valid and their providers are configured
- Check OpenClaw logs for the specific error:
  ```bash
  openclaw logs --tail 50
  ```
- Fallbacks trigger on: timeouts, 429 (rate limit), 500+ (server errors), connection refused
- Fallbacks do NOT trigger on: 400 (bad request), 401 (auth failure) -- these indicate config problems, not transient failures

### Slow responses from local models

**Cause:** Model is too large for your hardware, or Ollama is not using GPU.

**Fix:**
- Check GPU utilization: `nvidia-smi` (Linux) or Activity Monitor (macOS)
- On Apple Silicon, ensure Ollama is using Metal (it does by default)
- Try a smaller model: `llama3.1:8b` instead of `llama3.1:70b`
- Reduce context window in the agent config if the model is swapping to disk

---

## Next Steps

- **[04 -- Multi-Agent](../04-multi-agent/)** -- Set up specialized agents that use the models you just configured
- **[05 -- Memory](../05-memory/)** -- Add long-term memory so your agents remember across conversations
- **[09 -- Production](../09-production/)** -- Monitor costs and set up usage alerts
