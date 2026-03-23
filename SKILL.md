---
name: digitalocean-gradient
description: >-
  Build and deploy apps with DigitalOcean Gradient AI inference.
  Provides authentication, available models (OpenAI, Anthropic, Meta, DeepSeek, Mistral, Qwen, and more),
  integration via cURL/OpenAI SDK/Gradient SDK, image and audio generation, AI agents,
  managed databases (PostgreSQL with pgvector, MongoDB), Spaces object storage,
  and deployment to DigitalOcean App Platform with continuous deployment from GitHub.
  Use when building or deploying AI-powered applications with DigitalOcean or Gradient.
---

# DigitalOcean Gradient AI

Two inference options:
- **Serverless inference** (below) — call foundation models via API, pay per token, no infrastructure to manage. Best for getting started and variable traffic.
- **Dedicated inference** — deploy models on dedicated GPUs for sustained high-throughput workloads. See [references/dedicated-inference.md](references/dedicated-inference.md).

## Serverless Inference

### Authentication

1. Create a model access key in the [DigitalOcean console](https://cloud.digitalocean.com) under Gradient > Serverless Inference
2. Set it as an environment variable:

```bash
export DIGITAL_OCEAN_MODEL_ACCESS_KEY="your-key-here"
```

3. Pass as a Bearer token in requests:

```
Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY
```

### Base URL

```
https://inference.do-ai.run/v1/
```

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

### Python — OpenAI SDK

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

### Python — Gradient SDK

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
| `anthropic-claude-4.6-sonnet` | Anthropic | Chat |
| `anthropic-claude-opus-4.6` | Anthropic | Chat |
| `anthropic-claude-4.5-haiku` | Anthropic | Chat |
| `openai-gpt-5.4` | OpenAI | Chat |
| `openai-gpt-5.2` | OpenAI | Chat |
| `openai-gpt-5-mini` | OpenAI | Chat |
| `openai-gpt-4o` | OpenAI | Chat |
| `openai-o3` | OpenAI | Chat |
| `openai-gpt-image-1` | OpenAI | Image |
| `llama3.3-70b-instruct` | Meta | Chat |
| `alibaba-qwen3-32b` | Alibaba | Chat |
| `kimi-k2.5` | Moonshot AI | Chat |
| `fal-ai/flux/schnell` | fal | Image (async) |
| `fal-ai/elevenlabs/tts/multilingual-v2` | fal | Audio (async) |

For the full catalog (30+ models) with max tokens, capabilities, and embedding models, see [references/models.md](references/models.md).

## Key endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/v1/models` | GET | List available models |
| `/v1/chat/completions` | POST | Chat completions |
| `/v1/responses` | POST | Text/multimodal responses with reasoning |
| `/v1/images/generations` | POST | Image generation |
| `/v1/async-invoke` | POST | Async image/audio generation (fal models) |

## Additional references

- **Dedicated inference**: [references/dedicated-inference.md](references/dedicated-inference.md) — deploy models on dedicated GPUs, provisioning, public/private endpoints
- **Full model catalog**: [references/models.md](references/models.md) — all models with max tokens, capabilities, caching, reasoning
- **Image and audio generation**: [references/image-and-audio.md](references/image-and-audio.md) — sync and async workflows, base64 decoding, fal models
- **AI agents**: [references/agents.md](references/agents.md) — persistent agents with knowledge bases
- **Databases**: [references/databases.md](references/databases.md) — PostgreSQL + pgvector, MongoDB, Gradient Knowledge Bases + OpenSearch, App Platform binding
- **Spaces object storage**: [references/spaces.md](references/spaces.md) — S3-compatible storage for generated images, audio, uploads; CDN; Python and Node.js examples
- **Droplets**: [references/droplets.md](references/droplets.md) — Linux VMs, GPU Droplets for ML, cloud-init setup, when to use vs App Platform
- **Deploy to App Platform**: [references/deploy-to-app-platform.md](references/deploy-to-app-platform.md) — Flask and Next.js Dockerfiles, CD from GitHub

## Official docs

- [Available models](https://docs.digitalocean.com/products/gradient-ai-platform/details/models/)
- [Serverless inference guide](https://docs.digitalocean.com/products/gradient-ai-platform/how-to/use-serverless-inference/)
