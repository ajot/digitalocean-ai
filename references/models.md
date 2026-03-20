# Available Models

Full catalog of models available on DigitalOcean Gradient. For the latest list, see the [official models reference](https://docs.digitalocean.com/products/gradient-ai-platform/details/models/).

## Chat models

### Anthropic

| Model ID | Max Output Tokens | Context | Notes |
|---|---|---|---|
| `anthropic-claude-4.6-sonnet` | 64K | 1M | Prompt caching, tool calling |
| `anthropic-claude-4.5-sonnet` | 64K | 1M | Prompt caching, tool calling |
| `anthropic-claude-sonnet-4` | 64K | 1M | Prompt caching, tool calling |
| `anthropic-claude-4.5-haiku` | 64K | — | Tool calling |
| `anthropic-claude-opus-4.6` | 128K | 1M | Prompt caching, tool calling |
| `anthropic-claude-opus-4.5` | 64K | — | Prompt caching, tool calling |
| `anthropic-claude-4.1-opus` | 32K | — | Prompt caching, tool calling |
| `anthropic-claude-opus-4` | 32K | — | Prompt caching, tool calling |

**Prompt caching (Anthropic):** Set `cache_control` on message content:

```json
{
  "cache_control": {"type": "ephemeral", "ttl": "5m"}
}
```

Response includes `cache_created_input_tokens` and `cache_read_input_tokens`.

**Reasoning (Anthropic):** Use the `reasoning` parameter:

```json
{
  "reasoning": {"effort": "high", "max_tokens": 1024}
}
```

Effort levels: `low`, `medium`, `high`, `max`.

### OpenAI

| Model ID | Max Output Tokens | Notes |
|---|---|---|
| `openai-gpt-5.4` | 128K | 1M context, caching, `/v1/responses` only |
| `openai-gpt-5.3-codex` | 128K | 400K context, caching |
| `openai-gpt-5.2` | 128K | Caching |
| `openai-gpt-5-2-pro` | 128K | Caching |
| `openai-gpt-5.1-codex-max` | 128K | Caching |
| `openai-gpt-5` | 128K | Caching |
| `openai-gpt-5-mini` | 128K | Caching |
| `openai-gpt-5-nano` | 128K | Caching |
| `openai-gpt-4.1` | 32K | Caching |
| `openai-gpt-4o` | 16K | Caching |
| `openai-gpt-4o-mini` | 16K | Caching |
| `openai-o1` | — | Caching, reasoning |
| `openai-o3` | — | Caching, reasoning |
| `openai-o3-mini` | — | Caching, reasoning |
| `openai-gpt-oss-120b` | 131K | 117B parameters |
| `openai-gpt-oss-20b` | 131K | 21B parameters |

All OpenAI models support tool/function calling.

**Prompt caching (OpenAI):** Requires prompts >=1024 tokens. Set `prompt_cache_retention`:

```json
{
  "prompt_cache_retention": "24h"
}
```

**Reasoning (OpenAI):** Use `reasoning_effort`:

```json
{
  "reasoning_effort": "high"
}
```

Values: `none`, `low`, `medium`, `high`, `max`.

### Meta

| Model ID | Parameters | Max Output Tokens |
|---|---|---|
| `llama3.3-70b-instruct` | 70B | 128K |
| `llama3-8b-instruct` | 8B | 128K |

### Other providers

| Model ID | Provider | Parameters | Max Output Tokens | Notes |
|---|---|---|---|---|
| `alibaba-qwen3-32b` | Alibaba | 32B | 40K | |
| `deepseek-r1-distill-llama-70b` | DeepSeek | 70B | 32K | Add guardrails recommended |
| `mistral-nemo-instruct-2407` | Mistral | 12B | 128K | |
| `minimax-m2.5` | MiniMax | 230B | 128K | Supports `/v1/chat/completions` and `/v1/responses` |
| `kimi-k2.5` | Moonshot AI | 1T | 32K | Supports both endpoints |
| `nvidia-nemotron-3-super-120b` | NVIDIA | 120B | — | Supports both endpoints |
| `glm-5` | Z.ai | 744B | 128K | `/v1/chat/completions` only |

## Image and audio models

| Model ID | Provider | Type |
|---|---|---|
| `openai-gpt-image-1` | OpenAI | Image (sync) |
| `fal-ai/flux/schnell` | fal | Image (async) |
| `fal-ai/fast-sdxl` | fal | Image (async) |
| `fal-ai/stable-audio-25/text-to-audio` | fal | Audio (async) |
| `fal-ai/elevenlabs/tts/multilingual-v2` | fal | TTS (async) |

For image and audio generation workflows, see [image-and-audio.md](image-and-audio.md).

## Embedding models

| Model ID | Provider | Parameters | Token Window |
|---|---|---|---|
| GTE Large (v1.5) | Alibaba | — | 8,192 |
| All-MiniLM-L6-v2 | UKP Lab | 22M | 256 |
| Multi-QA-mpnet-base-dot-v1 | UKP Lab | 109M | 512 |
| Qwen3 Embedding 0.6B | Alibaba | 600M | 8,000 |
