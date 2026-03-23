# DigitalOcean Gradient — Agent Skill

An [Agent Skill](https://agentskills.io/) that teaches coding agents how to build and deploy apps with [DigitalOcean Gradient](https://docs.digitalocean.com/products/gradient-ai-platform/) AI inference.

Works with Claude Code, Cursor, Codex, and other skills-compatible agents.

## Install

```bash
npx skills add ajot/digitalocean-gradient
```

## What's included

| File | Description |
|---|---|
| `SKILL.md` | Serverless inference quick start (cURL, OpenAI SDK, Gradient SDK), model overview, key endpoints |
| `references/dedicated-inference.md` | Dedicated GPU inference for high-throughput production workloads |
| `references/models.md` | Full catalog of 30+ models — Anthropic, OpenAI, Meta, DeepSeek, Mistral, and more |
| `references/image-and-audio.md` | Sync and async generation workflows for images and audio |
| `references/agents.md` | Gradient AI Agents API with knowledge base support |
| `references/databases.md` | PostgreSQL + pgvector, MongoDB, Gradient Knowledge Bases + OpenSearch |
| `references/spaces.md` | S3-compatible object storage for generated images, audio, and assets with CDN |
| `references/droplets.md` | Linux VMs, GPU Droplets for ML, cloud-init setup |
| `references/deploy-to-app-platform.md` | Deploy to DO App Platform — Flask and Next.js templates with continuous deployment |

## Prerequisites

You'll need a DigitalOcean account with a [Gradient model access key](https://cloud.digitalocean.com).

## Official docs

- [Available models](https://docs.digitalocean.com/products/gradient-ai-platform/details/models/)
- [Serverless inference guide](https://docs.digitalocean.com/products/gradient-ai-platform/how-to/use-serverless-inference/)
- [Dedicated inference guide](https://docs.digitalocean.com/products/inference-hub/how-to/use-dedicated-inference)
