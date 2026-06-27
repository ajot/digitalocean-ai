---
name: digitalocean-ai
description: >-
  Build and deploy apps with the DigitalOcean Inference Engine.
  Provides authentication, available models (OpenAI, Anthropic, Meta, DeepSeek, Qwen, NVIDIA, and more),
  integration via cURL and the OpenAI SDK, image/audio/video generation, AI agents with knowledge bases,
  the Model Playground, batch inference, the Inference Router, evaluations,
  managed databases (PostgreSQL with pgvector, MongoDB), Spaces object storage,
  and deployment to DigitalOcean App Platform with continuous deployment from GitHub.
  Use when building or deploying AI-powered applications on DigitalOcean.
---

# DigitalOcean Inference

The DigitalOcean **Inference Engine** runs 70+ foundation models — every model, every modality (text, image, audio, video) — on one OpenAI-compatible API key.

Two inference options:
- **Serverless inference** (below) — call foundation models via API, pay per token, no infrastructure to manage. Best for getting started and variable traffic.
- **Dedicated inference** — deploy models on dedicated GPUs for sustained high-throughput workloads. See [references/dedicated-inference.md](references/dedicated-inference.md).

## Serverless Inference

### Authentication

1. Create a model access key in the [DigitalOcean console](https://cloud.digitalocean.com): click **INFERENCE** in the left menu, then **Manage**, then **Create model access key**. The secret is shown only once — copy it immediately.
2. Set it as an environment variable:

```bash
export DIGITAL_OCEAN_MODEL_ACCESS_KEY="your-key-here"
```

3. Pass as a Bearer token in requests:

```
Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY
```

One key unlocks every model and every modality.

### Base URL

```
https://inference.do-ai.run/v1/
```

Endpoints are **OpenAI- and Anthropic-compatible** — the OpenAI SDK, LangChain, and LlamaIndex work by pointing `base_url` at DigitalOcean with no other code changes.

## Quick start

### cURL

```bash
curl -s -X POST https://inference.do-ai.run/v1/chat/completions \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.3-70b-instruct",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_completion_tokens": 256
  }'
```

### Python — OpenAI SDK (recommended)

```bash
pip install openai python-dotenv
```

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(
    base_url="https://inference.do-ai.run/v1/",
    api_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY")
)

response = client.chat.completions.create(
    model="llama3.3-70b-instruct",
    messages=[{"role": "user", "content": "Hello!"}],
    max_completion_tokens=256
)

print(response.choices[0].message.content)
```

The OpenAI SDK is the primary, recommended integration path. To switch models, change the `model` field — nothing else changes.

### Try it with zero code: the Model Playground

The fastest first 60 seconds is the **Model Playground** — test and compare text, image, audio, and video models side by side in the browser, tune parameters, then copy a ready-to-run API request. Open it from the console: **INFERENCE > Model Catalog**, pick a model, then **Launch Playground**. See [references/playground.md](references/playground.md).

### Python — Gradient SDK (legacy)

> **Note:** The native Gradient SDK is slated for deprecation. Prefer the OpenAI SDK above for new code. The package is still named `gradient` (a literal package identifier, not the product name).

```bash
pip install gradient python-dotenv
```

```python
import os
from gradient import Gradient
from dotenv import load_dotenv

load_dotenv()

client = Gradient(model_access_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY"))

response = client.chat.completions.create(
    model="llama3.3-70b-instruct",
    messages=[{"role": "user", "content": "Hello!"}],
    max_completion_tokens=256
)

print(response.choices[0].message.content)
```

## Available models (overview)

| Model ID | Provider | Type |
|---|---|---|
| `anthropic-claude-opus-4.8` | Anthropic | Chat |
| `anthropic-claude-4.6-sonnet` | Anthropic | Chat |
| `anthropic-claude-haiku-4.5` | Anthropic | Chat |
| `openai-gpt-5.5` | OpenAI | Chat |
| `openai-gpt-5.4-mini` | OpenAI | Chat |
| `openai-o3` | OpenAI | Chat (reasoning) |
| `openai-gpt-image-2` | OpenAI | Image (sync) |
| `llama3.3-70b-instruct` | Meta | Chat |
| `deepseek-3.2` | DeepSeek | Chat |
| `kimi-k2.6` | Moonshot AI | Chat |
| `nvidia-nemotron-3-super-120b` | NVIDIA | Chat |
| `fal-ai/flux/schnell` | fal | Image (async) |
| `fal-ai/elevenlabs/tts/multilingual-v2` | fal | TTS (async) |
| `wan2-2-t2v-a14b` | — | Text-to-video (async) |

For the full catalog (70+ models) with max tokens, capabilities, embeddings, and preview status, see [references/models.md](references/models.md).

## Key endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/v1/models` | GET | List available (synchronous) models |
| `/v1/chat/completions` | POST | Chat completions (OpenAI-compatible) |
| `/v1/responses` | POST | Text/multimodal responses with reasoning |
| `/v1/images/generations` | POST | Sync image generation (OpenAI image models) |
| `/v1/async-invoke` | POST | Async image/audio/TTS generation (fal models) |
| `/v1/videos` | POST | Text-to-video (Wan 2.2); poll `/v1/videos/{id}`, fetch `/v1/videos/{id}/content` |
| `/v1/batches` | POST | Batch inference jobs (async, up to ~50% cheaper) |

## Additional references

- **Model Playground**: [references/playground.md](references/playground.md) — zero-code testing and comparison in the browser
- **Dedicated inference**: [references/dedicated-inference.md](references/dedicated-inference.md) — deploy models on dedicated GPUs, Bring Your Own Model, public/private endpoints
- **Full model catalog**: [references/models.md](references/models.md) — all models with max tokens, capabilities, caching, reasoning, embeddings
- **Image, audio & video generation**: [references/image-and-audio.md](references/image-and-audio.md) — sync and async workflows, base64 decoding, fal models
- **Batch, Router & Evaluations**: [references/batch-router-evals.md](references/batch-router-evals.md) — batch inference, the Inference Router (preview), and Evaluations (preview)
- **AI agents**: [references/agents.md](references/agents.md) — persistent agents with knowledge bases (RAG)
- **Databases**: [references/databases.md](references/databases.md) — PostgreSQL + pgvector, MongoDB, Knowledge Bases + OpenSearch, App Platform binding
- **Spaces object storage**: [references/spaces.md](references/spaces.md) — S3-compatible storage for generated images, audio, uploads; CDN; Python and Node.js examples
- **Droplets**: [references/droplets.md](references/droplets.md) — Linux VMs, GPU Droplets for ML, cloud-init setup, when to use vs App Platform
- **Deploy to App Platform**: [references/deploy-to-app-platform.md](references/deploy-to-app-platform.md) — Flask and Next.js Dockerfiles, CD from GitHub

## Official docs

- [Inference overview](https://docs.digitalocean.com/products/inference/)
- [Available models](https://docs.digitalocean.com/products/inference/details/models/)
- [Serverless inference guide](https://docs.digitalocean.com/products/inference/how-to/use-serverless-inference/)
- [Model Playground](https://docs.digitalocean.com/products/inference/how-to/use-model-playground/)
