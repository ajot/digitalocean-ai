# Deploy to DigitalOcean App Platform

Templates for deploying Gradient-powered apps to [DigitalOcean App Platform](https://docs.digitalocean.com/products/app-platform/) with continuous deployment from GitHub.

## Flask (Python)

### Project structure

```
my-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .env
```

### app.py

```python
import os
from flask import Flask, request, jsonify
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)

client = OpenAI(
    base_url="https://inference.do-ai.run/v1/",
    api_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY")
)

@app.route("/")
def index():
    return "Hello from Gradient!"

@app.route("/chat", methods=["POST"])
def chat():
    data = request.get_json()
    response = client.chat.completions.create(
        model=data.get("model", "llama3.3-70b-instruct"),
        messages=data["messages"],
        max_completion_tokens=data.get("max_tokens", 256)
    )
    return jsonify({"response": response.choices[0].message.content})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

### requirements.txt

```
flask
gunicorn
openai
python-dotenv
```

### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]
```

## Next.js (Node)

### Project setup

```bash
npx create-next-app@latest my-app
cd my-app
npm install openai
```

### next.config.js

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: "standalone",
};

module.exports = nextConfig;
```

`output: "standalone"` is required for App Platform deployment.

### API route (app/api/chat/route.ts)

```typescript
import { NextRequest, NextResponse } from "next/server";
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://inference.do-ai.run/v1/",
  apiKey: process.env.DIGITAL_OCEAN_MODEL_ACCESS_KEY,
});

export async function POST(req: NextRequest) {
  const { messages, model = "llama3.3-70b-instruct" } = await req.json();

  const response = await client.chat.completions.create({
    model,
    messages,
    max_completion_tokens: 256,
  });

  return NextResponse.json({
    response: response.choices[0].message.content,
  });
}
```

### Dockerfile

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV PORT=8080
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 8080
CMD ["node", "server.js"]
```

## GitHub repo setup

```bash
# Initialize and push to GitHub
git init
git add .
git commit -m "Initial commit"
gh repo create my-app --public --source=. --remote=origin --push
```

## Deploy to App Platform

### Environment variables

Set in the App Platform console or app spec:

| Variable | Value |
|---|---|
| `DIGITAL_OCEAN_MODEL_ACCESS_KEY` | Your model access key |

### App spec (YAML)

```yaml
name: my-gradient-app
services:
  - name: web
    github:
      repo: your-username/my-app
      branch: main
      deploy_on_push: true
    dockerfile_path: Dockerfile
    http_port: 8080
    instance_count: 1
    instance_size_slug: basic-xxs
    envs:
      - key: DIGITAL_OCEAN_MODEL_ACCESS_KEY
        scope: RUN_TIME
        type: SECRET
```

### Continuous deployment

With `deploy_on_push: true` in the app spec, every push to `main` triggers an automatic deployment.

## Deployment rules

- Port **must** be 8080 — both in the app code and Dockerfile
- Flask: include `gunicorn` in `requirements.txt`
- Next.js: set `output: "standalone"` in `next.config.js`
- Do not deploy until explicitly instructed
- Verify a GitHub repo exists before first deploy; create one with `gh repo create` if needed
- When polling deployment status, wait 10 seconds between checks
- Always confirm with the user before deleting an app
