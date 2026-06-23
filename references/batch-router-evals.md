# Batch Inference, Inference Router & Evaluations

Three ways to run inference smarter: process large jobs cheaply (Batch), route each request to the best-fit model automatically (Router), and score model quality against your own data (Evaluations).

> **Auth note:** Batch and Evaluations differ in which token they use. Batch and serverless calls use your **model access key** (`DIGITAL_OCEAN_MODEL_ACCESS_KEY`). The Evaluations control-plane API uses a **DigitalOcean API token** (`DIGITALOCEAN_TOKEN`). These are not interchangeable.

## Batch Inference

Run up to 50,000 requests as one asynchronous job with a 24-hour completion window, up to ~50% cheaper than real-time. Input is a JSONL file (one request per line, max 200 MB).

Each line of the input file is one request with a unique `custom_id`:

```json
{"custom_id": "req-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "llama3.3-70b-instruct", "messages": [{"role": "user", "content": "Hello!"}]}}
```

### 1. Upload the input file (presigned URL flow)

```bash
# Request a presigned upload URL
curl -s -X POST https://inference.do-ai.run/v1/batches/files \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file_name": "batch_requests.jsonl"}'
# → returns file_id + upload_url

# PUT the raw file bytes to the returned upload_url
curl -s -X PUT "<upload_url>" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @batch_requests.jsonl
```

### 2. Create the batch job

```bash
curl -s -X POST https://inference.do-ai.run/v1/batches \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "file_id": "FILE_ID_FROM_STEP_1",
    "provider": "openai",
    "endpoint": "/v1/chat/completions",
    "completion_window": "24h",
    "request_id": "YOUR_IDEMPOTENCY_UUID"
  }'
```

- `provider`: `openai` or `anthropic`.
- `endpoint`: required for OpenAI (`/v1/chat/completions` or `/v1/responses`). **Omit entirely for Anthropic models.**
- `completion_window`: only `24h` is accepted.
- Finish the file upload before creating the batch — the create call runs a HEAD check and fails until the JSONL is in place.

The response returns a `batch_id` and a `status` (initially `in_progress`).

## Inference Router (public preview)

The Router sends each request to the best-fit model by a `cheapest` or `fastest` policy, with automatic fallback (~200ms overhead). You create a router (with its selection policy) in the console or via the control-plane API, then call it by name.

Invoke it through the normal chat completions endpoint, with the router name prefixed by `router:` in the `model` field:

```bash
curl -s -X POST https://inference.do-ai.run/v1/chat/completions \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "router:my-router",
    "messages": [{"role": "user", "content": "Summarize this in one line."}]
  }'
```

- The response `model` field shows which model actually served the request.
- The matched route appears in the response header `x-model-router-selected-route`.
- Add `-H "X-Model-Affinity: session-001"` to pin a session: the first request routes normally and later same-ID requests reuse that model (skipping routing).

## Evaluations (public preview)

LLM-as-a-Judge scoring of models, routers, and deployments against your own dataset (CSV or JSONL, max 1 GB / 1,000 rows). Uses the control-plane API at `api.digitalocean.com` with a DigitalOcean API token.

### 1. Upload a dataset

```bash
curl -s -X POST \
  https://api.digitalocean.com/v2/gen-ai/model_evaluation/datasets/file_upload_presigned_urls \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"files": [{"file_name": "dataset.jsonl", "file_size": 2048}]}'
```

### 2. Create an evaluation run

Get candidate/judge model UUIDs from `GET /v2/gen-ai/models/catalog` and metric UUIDs from `GET /v2/gen-ai/model_evaluation_metrics`.

```bash
curl -s -X POST https://api.digitalocean.com/v2/gen-ai/model_evaluation_runs \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "opus-vs-baseline",
    "candidate_model_uuid": "CANDIDATE_MODEL_UUID",
    "judge_model_uuid": "JUDGE_MODEL_UUID",
    "dataset_uuid": "DATASET_UUID",
    "metric_uuids": ["METRIC_UUID"]
  }'
```

### 3. Download results

```bash
curl -s \
  "https://api.digitalocean.com/v2/gen-ai/model_evaluation_runs/<eval_run_uuid>/results/download_url" \
  -H "Authorization: Bearer $DIGITALOCEAN_TOKEN"
```

Judge model options include Claude Opus, GPT-5.x, Claude Sonnet, and others from the catalog.

## Official docs

- [Use batch inference](https://docs.digitalocean.com/products/inference/how-to/use-batch-inference/)
- [Use the Inference Router](https://docs.digitalocean.com/products/inference/how-to/use-inference-router/)
- [Evaluate models](https://docs.digitalocean.com/products/inference/how-to/evaluate-models/)
