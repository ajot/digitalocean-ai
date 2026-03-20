# Gradient AI Agents

Agents are persistent AI assistants with attached knowledge bases. Unlike serverless inference (stateless per-request), agents maintain context and can reference uploaded documents.

## Setup

Agents require separate credentials from serverless inference:

```bash
export AGENT_ENDPOINT="https://agents.do-ai.run/v1/your-agent-id"
export AGENT_ACCESS_KEY="your-agent-access-key"
```

Create agents and get credentials in the [DigitalOcean console](https://cloud.digitalocean.com) under Gradient > Agents.

## Base URL

```
https://agents.do-ai.run/v1/
```

## Chat with an agent

### cURL

```bash
curl -s -X POST "$AGENT_ENDPOINT/chat/completions" \
  -H "Authorization: Bearer $AGENT_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "What can you help me with?"}]
  }'
```

### Python — OpenAI SDK

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(
    base_url=os.getenv("AGENT_ENDPOINT"),
    api_key=os.getenv("AGENT_ACCESS_KEY")
)

response = client.chat.completions.create(
    model="n/a",
    messages=[{"role": "user", "content": "What can you help me with?"}]
)

print(response.choices[0].message.content)
```

Note: The `model` parameter is required by the SDK but ignored by the agent endpoint — the agent uses its configured model.

### Python — Gradient SDK

```python
import os
from gradient import Gradient
from dotenv import load_dotenv

load_dotenv()

client = Gradient(
    base_url=os.getenv("AGENT_ENDPOINT"),
    agent_access_key=os.getenv("AGENT_ACCESS_KEY")
)

response = client.chat.completions.create(
    model="n/a",
    messages=[{"role": "user", "content": "What can you help me with?"}]
)

print(response.choices[0].message.content)
```
