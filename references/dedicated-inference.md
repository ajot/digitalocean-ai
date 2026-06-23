# Dedicated Inference

Deploy models on dedicated GPUs for sustained, high-throughput workloads. Billed per GPU-hour (not per token like serverless).

## When to use dedicated vs serverless

| | Serverless | Dedicated |
|---|---|---|
| Billing | Per token | Per GPU-hour |
| Setup | None — just call the API | Provision a deployment |
| Traffic pattern | Variable / spiky | Consistent / high-throughput |
| Models | Pre-selected catalog | Open-source from Hugging Face + catalog |
| Customization | None | GPU selection, performance tuning |
| Best for | Getting started, prototypes, hackathons | Production workloads, custom models |

## Provision a deployment

Create via the [DO Console](https://cloud.digitalocean.com) under INFERENCE > Dedicated Inference > Deploy.

### Console steps

1. Select region (ATL1, NYC2, or TOR1)
2. Choose a model from the catalog (or specify a Hugging Face model slug)
3. Select GPU plan (1 or 8 GPU — AMD MI300X or NVIDIA options)
4. Set node count
5. Name the deployment (lowercase, alphanumeric, hyphens)
6. Deploy

The auth token is shown **only once** after creation — copy it immediately.

### API

```bash
curl -X POST "https://api.digitalocean.com/v2/dedicated-inferences" \
  -H "Authorization: Bearer $DO_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "spec": {
      "name": "my-inference",
      "region": "atl1",
      "model_deployments": [{
        "model_slug": "mistral/mistral-7b-instruct-v3",
        "accelerator_slug": "gpu-mi300x1-192gb"
      }]
    }
  }'
```

For gated Hugging Face models, include `"hugging_face_token": "your-hf-token"`.

Provisioning takes a few minutes. Status goes from `Provisioning` → `Active`.

## Use the endpoint

Each deployment gets a public and private endpoint URL. Find them in the Console under your deployment's details.

### Chat completions

```bash
curl -X POST "https://<your-endpoint>/v1/chat/completions" \
  -H "Authorization: Bearer $DEDICATED_INFERENCE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral/mistral-7b-instruct-v3",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 256
  }'
```

### Python — OpenAI SDK

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://<your-endpoint>/v1/",
    api_key=os.getenv("DEDICATED_INFERENCE_TOKEN")
)

response = client.chat.completions.create(
    model="mistral/mistral-7b-instruct-v3",
    messages=[{"role": "user", "content": "Hello!"}],
    max_tokens=256
)

print(response.choices[0].message.content)
```

The OpenAI SDK works the same way as serverless — just change the `base_url` and `api_key` to your dedicated endpoint and token.

## Public vs private endpoints

- **Public endpoint**: Accessible from the internet. Use for development and external-facing apps.
- **Private endpoint**: Accessible only within your VPC. Use for production apps running on Droplets or App Platform in the same region. Requires downloading the CA certificate for TLS verification:

```bash
curl --cacert /path/to/ca-cert.pem \
  -X POST "https://<private-endpoint>/v1/chat/completions" \
  -H "Authorization: Bearer $DEDICATED_INFERENCE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"model": "...", "messages": [...]}'
```

## Manage deployments

### API

```bash
# List deployments
curl -X GET "https://api.digitalocean.com/v2/dedicated-inferences" \
  -H "Authorization: Bearer $DO_API_TOKEN"

# Get deployment details
curl -X GET "https://api.digitalocean.com/v2/dedicated-inferences/<id>" \
  -H "Authorization: Bearer $DO_API_TOKEN"

# Delete deployment
curl -X DELETE "https://api.digitalocean.com/v2/dedicated-inferences/<id>" \
  -H "Authorization: Bearer $DO_API_TOKEN"
```

## Official docs

- [Dedicated inference guide](https://docs.digitalocean.com/products/inference/how-to/use-dedicated-inference/)
- [GPU Droplets](https://docs.digitalocean.com/products/gpu-droplets/)
