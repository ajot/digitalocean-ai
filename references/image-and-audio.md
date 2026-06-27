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

## Async generation (fal models)

Fal models use an async workflow: **submit** a request, **poll** for status, **retrieve** the result. The submit body uses `model_id` (not `model`). Video generation does **not** use this endpoint — see [Video generation](#video-generation-wan-22) below.

### Available fal models

| Model ID | Type |
|---|---|
| `fal-ai/flux/schnell` | Image generation |
| `fal-ai/fast-sdxl` | Image generation |
| `fal-ai/stable-audio-25/text-to-audio` | Audio generation |
| `fal-ai/elevenlabs/tts/multilingual-v2` | Text-to-speech |

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
```

## Video generation (Wan 2.2)

Text-to-video uses a **separate** endpoint, `/v1/videos` — NOT `/v1/async-invoke`. The shape differs from the fal async flow: it uses `model` (not `model_id`) and a top-level `prompt` (not nested under `input`). Generation takes ~30s to several minutes. Verified working with `wan2-2-t2v-a14b` (produces an H.264 MP4).

### Step 1: Submit

```bash
curl -s -X POST https://inference.do-ai.run/v1/videos \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "wan2-2-t2v-a14b",
    "prompt": "A neon city skyline at dusk, slow cinematic camera pan",
    "size": "1280x720",
    "fps": 16
  }'
# → {"id": "video_...", "status": "queued"}
```

### Step 2: Poll

```bash
curl -s https://inference.do-ai.run/v1/videos/VIDEO_ID \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY"
# status: queued → in_progress → completed
```

> The poll path is `/v1/videos/{id}`. (Some docs show `/v1/video/generations/{id}`, which 404s — don't use it.)

### Step 3: Download the MP4

When `status` is `completed`, the status body's `output` is `null` — fetch the binary from the `/content` endpoint instead:

```bash
curl -s https://inference.do-ai.run/v1/videos/VIDEO_ID/content \
  -H "Authorization: Bearer $DIGITAL_OCEAN_MODEL_ACCESS_KEY" \
  --output video.mp4
```

> **Retry the `/content` call.** It intermittently returns HTTP 500 (a server-side SSL error reaching DigitalOcean's storage) even on a completed job, then succeeds on retry.

### Python helper for video

```python
import os
import time
import requests

BASE_URL = "https://inference.do-ai.run/v1"
HEADERS = {"Authorization": f"Bearer {os.getenv('DIGITAL_OCEAN_MODEL_ACCESS_KEY')}"}

def generate_video(prompt, size="1280x720", fps=16, poll_interval=10):
    job = requests.post(f"{BASE_URL}/videos", headers={**HEADERS, "Content-Type": "application/json"}, json={
        "model": "wan2-2-t2v-a14b", "prompt": prompt, "size": size, "fps": fps
    }).json()
    vid = job["id"]

    while True:
        status = requests.get(f"{BASE_URL}/videos/{vid}", headers=HEADERS).json()["status"]
        if status == "completed":
            break
        if status == "failed":
            raise RuntimeError("video generation failed")
        time.sleep(poll_interval)

    # /content is intermittently 500 — retry a few times
    for _ in range(5):
        r = requests.get(f"{BASE_URL}/videos/{vid}/content", headers=HEADERS)
        if r.ok and r.headers.get("content-type", "").startswith("video/"):
            with open("video.mp4", "wb") as f:
                f.write(r.content)
            return "video.mp4"
        time.sleep(3)
    raise RuntimeError("could not fetch video content after retries")

generate_video("A neon city skyline at dusk, slow cinematic camera pan")
```
