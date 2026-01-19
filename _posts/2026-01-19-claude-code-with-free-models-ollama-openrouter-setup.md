---
layout: post
title: "Claude Code + Ollama/OpenRouter: Fix \"model not found\" (set 3 default models)"
date: 2026-01-19
categories: [AI, security, automation]
tags: [claude-code, ollama, openrouter, AI, automation, LLM]
---

**TL;DR**: Claude Code internally uses **3 default models** (Haiku/Sonnet/Opus). If you only configure one, you’ll hit intermittent 404 `model ... not found` errors. Fix: set `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `ANTHROPIC_DEFAULT_SONNET_MODEL`, and `ANTHROPIC_DEFAULT_OPUS_MODEL`.

![Claude Code](/img/20260119_claude-code.png)

## Introduction

Recently, Ollama added support for Claude Code. In the [Ollama Blog](https://ollama.com/blog/claude), the instructions are clear and easy:

```bash
export ANTHROPIC_AUTH_TOKEN=ollama
export ANTHROPIC_BASE_URL=http://localhost:11434

# Run Claude Code with an Ollama model:
claude --model gpt-oss:20b

# Models in Ollama's Cloud also work with Claude Code:
claude --model glm-4.7:cloud
```

However, if you use a proxy server, you might see the following error:

**HTTP Request:**

```http
{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 1,
  "messages": [
    {
      "role": "user",
      "content": "count"
    }
  ],
  "tools":[
    {
      "name":"Task",
<...snip...>
```

**HTTP Response:**

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "type": "error",
  "error": {
    "type": "not_found_error",
    "message": "model 'claude-haiku-4-5-20251001' not found"
  }
}
```

In addition, telemetry is continuously sending requests to `http://localhost:11434/api/event_logging/batch`, getting a 404. My TOC doesn't accept this :D

## The Problem Nobody Tells You About

Claude Code doesn't use a single model, it uses 3 internally:

| Tier | Variable | Purpose |
|------|----------|---------|
| **HAIKU** | `ANTHROPIC_DEFAULT_HAIKU_MODEL` | Fast auxiliary tasks: terminal titles, conversation summaries, bash path extraction |
| **SONNET** | `ANTHROPIC_DEFAULT_SONNET_MODEL` | Main coding model |
| **OPUS** | `ANTHROPIC_DEFAULT_OPUS_MODEL` | Complex reasoning and deep analysis |

If you don't configure all 3, Claude Code will fail with 404 errors looking for the other models, for instance `claude-haiku-4-5-20251001`. This will happen depending on the internal task performed by Claude Code.

## Working with Ollama (Local)

It's highly recommended to bump the default context length to 64K and allow multiple loaded models + parallel requests:

```bash
export OLLAMA_CONTEXT_LENGTH=64000
export OLLAMA_MAX_LOADED_MODELS=3
export OLLAMA_NUM_PARALLEL=2
```

### One-liner for 64GB RAM (Full Power)

If you want maximum performance and have the RAM to spare:

```bash
DISABLE_TELEMETRY=1 \
ANTHROPIC_AUTH_TOKEN=ollama \
ANTHROPIC_BASE_URL=http://localhost:11434 \
ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:8b" \
ANTHROPIC_DEFAULT_SONNET_MODEL="qwen3-coder:latest" \
ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:72b-q4_K_M" \
claude
```

| Tier | Model | RAM | Notes |
|------|-------|-----|-------|
| HAIKU | `qwen3:8b` | ~5GB | Fast for auxiliary tasks |
| SONNET | `qwen3-coder:latest` | ~20GB | Code specialized |
| OPUS | `qwen3:72b-q4_K_M` | ~45GB | Full power reasoning |

### One-liner for 64GB RAM (Lightweight)

If you prefer to keep RAM free for other tasks:

```bash
DISABLE_TELEMETRY=1 \
ANTHROPIC_AUTH_TOKEN=ollama \
ANTHROPIC_BASE_URL=http://localhost:11434 \
ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
ANTHROPIC_DEFAULT_SONNET_MODEL="qwen3-coder:latest" \
ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:32b-q4_K_M" \
claude
```

| Tier | Model | RAM | Notes |
|------|-------|-----|-------|
| HAIKU | `qwen3:4b` | ~3GB | Minimal footprint |
| SONNET | `qwen3-coder:latest` | ~20GB | Code specialized |
| OPUS | `qwen3:32b-q4_K_M` | ~20GB | Good reasoning, less RAM |

### One-liner for 32GB RAM (Full Power)

Pushing 32GB to the limit:

```bash
DISABLE_TELEMETRY=1 \
ANTHROPIC_AUTH_TOKEN=ollama \
ANTHROPIC_BASE_URL=http://localhost:11434 \
ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-coder-v2:16b" \
ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:32b-q4_K_M" \
claude
```

| Tier | Model | RAM | Notes |
|------|-------|-----|-------|
| HAIKU | `qwen3:4b` | ~3GB | Minimal footprint |
| SONNET | `deepseek-coder-v2:16b` | ~10GB | Great coder, lighter |
| OPUS | `qwen3:32b-q4_K_M` | ~20GB | Best reasoning at this size |

### One-liner for 32GB RAM (Lightweight)

If you want to keep your system responsive:

```bash
DISABLE_TELEMETRY=1 \
ANTHROPIC_AUTH_TOKEN=ollama \
ANTHROPIC_BASE_URL=http://localhost:11434 \
ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-coder-v2:16b" \
ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-coder-v2:16b" \
claude
```

| Tier | Model | RAM | Notes |
|------|-------|-----|-------|
| HAIKU | `qwen3:4b` | ~3GB | Minimal footprint |
| SONNET | `deepseek-coder-v2:16b` | ~10GB | Great coder |
| OPUS | `deepseek-coder-v2:16b` | ~10GB | Same model, less context switching |

This last config uses the same model for SONNET and OPUS. You lose some reasoning power, but it's much lighter on RAM and avoids constant model swapping.

If you're using Ollama Cloud, there are other powerful alternatives available. Check the cloud models [here](https://ollama.com/search?c=cloud).

## OpenRouter (Free Models)

### One-liner

```bash
DISABLE_TELEMETRY=1 \
ANTHROPIC_BASE_URL="https://openrouter.ai/api" \
ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
ANTHROPIC_API_KEY="" \
ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen/qwen3-4b:free" \
ANTHROPIC_DEFAULT_SONNET_MODEL="qwen/qwen3-coder:free" \
ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek/deepseek-r1-0528:free" \
claude
```

Important: `ANTHROPIC_API_KEY=""` must be explicitly empty (not just unset) for OpenRouter to work.

### Recommended Free Models

| Tier | Model | Context | Notes |
|------|-------|---------|-------|
| HAIKU | `qwen/qwen3-4b:free` | 40K | Light and fast |
| HAIKU | `xiaomi/mimo-v2-flash:free` | 262K | More context |
| SONNET | `qwen/qwen3-coder:free` | 262K | 480B MoE, excellent for code |
| SONNET | `mistralai/devstral-2512:free` | 262K | New Devstral |
| OPUS | `deepseek/deepseek-r1-0528:free` | 163K | Reasoning chains |
| OPUS | `meta-llama/llama-3.1-405b-instruct:free` | 131K | 405B parameters |

## Shell Aliases

Add these to your `.bashrc` or `.zshrc`:

### Ollama 64GB (Full Power)

```bash
alias claude-ollama='DISABLE_TELEMETRY=1 \
  ANTHROPIC_AUTH_TOKEN=ollama \
  ANTHROPIC_BASE_URL=http://localhost:11434 \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:8b" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="qwen3-coder:latest" \
  ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:72b-q4_K_M" \
  claude'
```

### Ollama 64GB (Lightweight)

```bash
alias claude-ollama-light='DISABLE_TELEMETRY=1 \
  ANTHROPIC_AUTH_TOKEN=ollama \
  ANTHROPIC_BASE_URL=http://localhost:11434 \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="qwen3-coder:latest" \
  ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:32b-q4_K_M" \
  claude'
```

### Ollama 32GB (Full Power)

```bash
alias claude-ollama='DISABLE_TELEMETRY=1 \
  ANTHROPIC_AUTH_TOKEN=ollama \
  ANTHROPIC_BASE_URL=http://localhost:11434 \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-coder-v2:16b" \
  ANTHROPIC_DEFAULT_OPUS_MODEL="qwen3:32b-q4_K_M" \
  claude'
```

### Ollama 32GB (Lightweight)

```bash
alias claude-ollama-light='DISABLE_TELEMETRY=1 \
  ANTHROPIC_AUTH_TOKEN=ollama \
  ANTHROPIC_BASE_URL=http://localhost:11434 \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen3:4b" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="deepseek-coder-v2:16b" \
  ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek-coder-v2:16b" \
  claude'
```

### OpenRouter

```bash
alias claude-openrouter='DISABLE_TELEMETRY=1 \
  ANTHROPIC_BASE_URL="https://openrouter.ai/api" \
  ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
  ANTHROPIC_API_KEY="" \
  ANTHROPIC_DEFAULT_HAIKU_MODEL="qwen/qwen3-4b:free" \
  ANTHROPIC_DEFAULT_SONNET_MODEL="qwen/qwen3-coder:free" \
  ANTHROPIC_DEFAULT_OPUS_MODEL="deepseek/deepseek-r1-0528:free" \
  claude'
```

## References

- [Ollama Blog: Claude Code support](https://ollama.com/blog/claude)
- [Claude Code Model Configuration](https://docs.anthropic.com/en/docs/claude-code/model-config)
- [OpenRouter Claude Code Integration](https://openrouter.ai/docs/guides/guides/claude-code-integration)
