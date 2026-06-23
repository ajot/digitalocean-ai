# Model Playground

The **Model Playground** is a zero-code, in-browser interface for trying and comparing foundation models before you wire them into an app. It's the fastest way to evaluate a model in the first 60 seconds.

## Open it

In the [DigitalOcean Control Panel](https://cloud.digitalocean.com): click **Model Catalog** under **INFERENCE** in the left menu, pick a model, then click **Launch Playground** on the model card.

## What you can do

- **Send prompts** and review responses interactively.
- **Compare models side by side** — click **Compare Another Model**, with a **Sync Inputs** toggle to share the same prompt and attachments across models.
- **Tune parameters** via the settings panel: temperature, Top P, token limits, and type-specific controls (image resolution/aspect ratio, video FPS/motion strength, audio duration/voice).
- **Upload images** from local storage for multimodal prompts.
- **Generate multimodal artifacts** — text, images, audio, and video, when the selected model supports them.
- **Export code** — the settings panel lets you copy the API request to reproduce the call in your own app.

## Supported modalities

Text, reasoning, image, audio, and video — whatever the chosen model supports.

## Billing

Playground usage requires confirming standard billing and is then charged per the model playground pricing (same per-token / per-generation rates as the API).

## Official docs

- [Use the Model Playground](https://docs.digitalocean.com/products/inference/how-to/use-model-playground/)
- [Browse the model catalog](https://docs.digitalocean.com/products/inference/how-to/browse-model-catalog/)
