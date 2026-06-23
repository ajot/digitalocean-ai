# Image, Audio & Video Generation

Two patterns: **sync** for OpenAI image models, **async** for fal models (image, audio, TTS) and video models.

For the full API reference, see the [serverless inference guide](https://docs.digitalocean.com/products/inference/how-to/use-serverless-inference/) and [use fal models](https://docs.digitalocean.com/products/inference/how-to/use-fal-models/).

## Sync image generation (OpenAI)

Use `/v1/images/generations` with an OpenAI image model such as `openai-gpt-image-2`.

### cURL

```bash
curl -s -X POST https://inference.do-ai.run/v1/images/generations \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai-gpt-image-2",
    "prompt": "A cute baby sea otter wearing a hat",
    "n": 1,
    "size": "1024x1024"
  }' | jq -r '.data[0].b64_json' | base64 --decode > image.png
```

### Python — OpenAI SDK

```python
import base64
from openai import OpenAI

client = OpenAI(
    base_url="https://inference.do-ai.run/v1/",
    api_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY")
)

response = client.images.generate(
    model="openai-gpt-image-2",
    prompt="A cute baby sea otter wearing a hat",
    n=1,
    size="1024x1024"
)

image_data = base64.b64decode(response.data[0].b64_json)
with open("image.png", "wb") as f:
    f.write(image_data)
```

> **Legacy:** The native Gradient SDK (`from gradient import Gradient`, `client.images.generate(...)`) also works but is slated for deprecation — prefer the OpenAI SDK above.

## Async generation (fal & video models)

Fal models and video models use an async workflow: **submit** a request, **poll** for status, **retrieve** the result. The submit body uses `model_id` (not `model`).

### Available async models

| Model ID | Type |
|---|---|
| `fal-ai/flux/schnell` | Image generation |
| `fal-ai/fast-sdxl` | Image generation |
| `fal-ai/stable-audio-25/text-to-audio` | Audio generation |
| `fal-ai/elevenlabs/tts/multilingual-v2` | Text-to-speech |
| `wan2-2-t2v-a14b` | Text-to-video |

### Step 1: Submit request

```bash
curl -s -X POST https://inference.do-ai.run/v1/async-invoke \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model_id": "fal-ai/flux/schnell",
    "input": {
      "prompt": "A futuristic cityscape at sunset"
    }
  }'
```

Response:

```json
{
  "request_id": "abc123",
  "status": "QUEUED"
}
```

### Step 2: Poll status

```bash
curl -s https://inference.do-ai.run/v1/async-invoke/REQUEST_ID/status \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY"
```

Possible statuses: `QUEUED`, `IN_PROGRESS`, `COMPLETED`.

### Step 3: Retrieve result

```bash
curl -s https://inference.do-ai.run/v1/async-invoke/REQUEST_ID \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY"
```

### Python helper for async workflow

```python
import os
import time
import requests

BASE_URL = "https://inference.do-ai.run/v1"
HEADERS = {
    "Authorization": f"Bearer {os.getenv('DIGITAL_OCEAN_MODEL_ACCESS_KEY')}",
    "Content-Type": "application/json"
}

def async_generate(model_id, input_data, poll_interval=2):
    """Submit an async request, poll until complete, return result."""
    # Submit
    resp = requests.post(f"{BASE_URL}/async-invoke", headers=HEADERS, json={
        "model_id": model_id,
        "input": input_data
    })
    request_id = resp.json()["request_id"]

    # Poll
    while True:
        status_resp = requests.get(
            f"{BASE_URL}/async-invoke/{request_id}/status",
            headers=HEADERS
        )
        status = status_resp.json()["status"]
        if status == "COMPLETED":
            break
        time.sleep(poll_interval)

    # Retrieve
    result = requests.get(
        f"{BASE_URL}/async-invoke/{request_id}",
        headers=HEADERS
    )
    return result.json()

# Image generation
result = async_generate("fal-ai/flux/schnell", {
    "prompt": "A futuristic cityscape at sunset"
})

# Text-to-speech
result = async_generate("fal-ai/elevenlabs/tts/multilingual-v2", {
    "text": "Hello from DigitalOcean Inference!",
    "voice_id": "JBFqnCBsd6RMkjVDRZzb"
})

# Audio generation
result = async_generate("fal-ai/stable-audio-25/text-to-audio", {
    "prompt": "Calm ambient music with soft piano"
})

# Text-to-video
result = async_generate("wan2-2-t2v-a14b", {
    "prompt": "A neon city skyline at dusk, slow camera pan"
})
```
