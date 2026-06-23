# DigitalOcean Inference — Agent Skill

An [Agent Skill](https://agentskills.io/) that teaches coding agents how to build and deploy apps with the [DigitalOcean Inference Engine](https://docs.digitalocean.com/products/inference/).

Works with Claude Code, Cursor, Codex, and other skills-compatible agents.

## Install

```bash
npx skills add ajot/digitalocean-ai
```

## What's included

| File | Description |
|---|---|
| `SKILL.md` | Serverless inference quick start (cURL, OpenAI SDK), model overview, key endpoints, Model Playground |
| `references/playground.md` | Zero-code model testing and comparison in the browser |
| `references/dedicated-inference.md` | Dedicated GPU inference for high-throughput production workloads, Bring Your Own Model |
| `references/models.md` | Full catalog of 70+ models — Anthropic, OpenAI, Meta, DeepSeek, Qwen, NVIDIA, and more |
| `references/image-and-audio.md` | Sync and async generation workflows for images, audio, and video |
| `references/batch-router-evals.md` | Batch inference, the Inference Router (preview), and Evaluations (preview) |
| `references/agents.md` | Agents API with knowledge base support (RAG) |
| `references/databases.md` | PostgreSQL + pgvector, MongoDB, Knowledge Bases + OpenSearch |
| `references/spaces.md` | S3-compatible object storage for generated images, audio, and assets with CDN |
| `references/droplets.md` | Linux VMs, GPU Droplets for ML, cloud-init setup |
| `references/deploy-to-app-platform.md` | Deploy to DO App Platform — Flask and Next.js templates with continuous deployment |

## Prerequisites

You'll need a DigitalOcean account with a model access key. Create one in the [console](https://cloud.digitalocean.com): **INFERENCE** in the left menu, then **Manage**, then **Create model access key**.

## Official docs

- [Inference overview](https://docs.digitalocean.com/products/inference/)
- [Available models](https://docs.digitalocean.com/products/inference/details/models/)
- [Serverless inference guide](https://docs.digitalocean.com/products/inference/how-to/use-serverless-inference/)
- [Dedicated inference guide](https://docs.digitalocean.com/products/inference/how-to/use-dedicated-inference/)
