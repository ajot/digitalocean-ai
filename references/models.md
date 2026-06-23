# Available Models

Full catalog of models available on the DigitalOcean Inference Engine. The catalog changes frequently — for the authoritative current list, see the [official models reference](https://docs.digitalocean.com/products/inference/details/models/). Models marked **Preview** are in public preview.

## Chat models

### Anthropic

| Model ID | Max Output Tokens | Context | Notes |
|---|---|---|---|
| `anthropic-claude-opus-4.8` | 128K | 1M | Prompt caching, tool calling |
| `anthropic-claude-opus-4.7` | 128K | 1M | Prompt caching, tool calling |
| `anthropic-claude-opus-4.6` | 128K | 1M | Prompt caching, tool calling |
| `anthropic-claude-opus-4.5` | 64K | — | Prompt caching, tool calling |
| `anthropic-claude-4.1-opus` | 32K | — | Prompt caching, tool calling |
| `anthropic-claude-4.6-sonnet` | 64K | 1M | Prompt caching, tool calling |
| `anthropic-claude-4.5-sonnet` | 64K | 1M | Prompt caching, tool calling |
| `anthropic-claude-haiku-4.5` | 64K | — | Tool calling |

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
| `openai-gpt-5.5` | 128K | Caching |
| `openai-gpt-5.4` | 128K | 1M context, caching, `/v1/responses` only |
| `openai-gpt-5.4-mini` | 128K | Caching |
| `openai-gpt-5.4-nano` | 128K | Caching |
| `openai-gpt-5.4-pro` | 128K | Caching |
| `openai-gpt-5.3-codex` | 128K | 400K context, caching |
| `openai-gpt-5.2` | 128K | Caching |
| `openai-gpt-5.2-pro` | 128K | Caching |
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
| `llama-4-maverick` | — | 128K |
| `llama3-8b-instruct` | 8B | 128K |

### Other providers

| Model ID | Provider | Notes |
|---|---|---|
| `alibaba-qwen3-32b` | Alibaba | Chat |
| `qwen3-coder-flash` | Alibaba | Code-focused |
| `qwen3.5-397b-a17b` | Alibaba | MoE |
| `deepseek-3.2` | DeepSeek | Chat |
| `deepseek-v4-pro` | DeepSeek | Chat |
| `deepseek-4-flash` | DeepSeek | Chat |
| `deepseek-r1-distill-llama-70b` | DeepSeek | Reasoning; add guardrails recommended |
| `kimi-k2.6` | Moonshot AI | Chat |
| `kimi-k2.5` | Moonshot AI | Chat |
| `nvidia-nemotron-3-super-120b` | NVIDIA | **Preview** |
| `nemotron-3-ultra-550b` | NVIDIA | Chat |
| `minimax-m2.5` | MiniMax | **Preview** |
| `mistral-3-14B` | Mistral | Chat |
| `gemma-4-31B-it` | Google | Chat |
| `glm-5.2` | Z.ai | Chat |
| `glm-5` | Z.ai | Chat |
| `arcee-trinity-large-thinking` | Arcee | **Preview**, reasoning |

## Image, audio & video models

| Model ID | Provider | Type |
|---|---|---|
| `openai-gpt-image-2` | OpenAI | Image (sync) |
| `openai-gpt-image-1.5` | OpenAI | Image (sync) |
| `openai-gpt-image-1` | OpenAI | Image (sync) |
| `stable-diffusion-3.5-large` | Stability AI | Image (sync) |
| `fal-ai/flux/schnell` | fal | Image (async) |
| `fal-ai/fast-sdxl` | fal | Image (async) |
| `fal-ai/stable-audio-25/text-to-audio` | fal | Audio (async) |
| `fal-ai/elevenlabs/tts/multilingual-v2` | fal | TTS (async) |
| `qwen3-tts-voicedesign` | Alibaba | TTS |
| `wan2-2-t2v-a14b` | — | Text-to-video (async) |

Sync image models use `/v1/images/generations`; fal and video models use the async `/v1/async-invoke` workflow. See [image-and-audio.md](image-and-audio.md).

## Embedding models

| Model | Provider | Token Window |
|---|---|---|
| GTE Large (v1.5) | Alibaba | 8,192 |
| BGE M3 | BAAI | 8,192 |
| Qwen3 Embedding 0.6B | Alibaba | 8,000 (**Preview**) |
| E5 Large (multilingual) | Microsoft | 512 |
| E5 Large (v2) | Microsoft | 512 |
| All-MiniLM-L6-v2 | UKP Lab | 256 |
| Multi-QA-mpnet-base-dot-v1 | UKP Lab | 512 |

Generate embeddings via `/v1/embeddings`. See [generate-vector-embeddings](https://docs.digitalocean.com/products/inference/how-to/generate-vector-embeddings/).

## Reranking

| Model | Provider |
|---|---|
| BGE Reranker (v2) M3 | BAAI |
