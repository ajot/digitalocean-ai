# Agents

Agents are persistent AI assistants with attached knowledge bases. Unlike serverless inference (stateless per-request), agents maintain context and can reference uploaded documents (RAG).

## Setup

Agents require separate credentials from serverless inference. Each deployed agent has its own endpoint and access key:

```bash
export AGENT_ENDPOINT="https://your-agent-id.agents.do-ai.run"
export AGENT_ACCESS_KEY="your-agent-access-key"
```

Create agents and manage credentials in the [DigitalOcean console](https://cloud.digitalocean.com): click **Agent Platform** under **INFERENCE** in the left menu, open the **Workspaces** tab, and create or select an agent. Create an access key to call the agent externally.

## Endpoint

Append `/api/v1/chat/completions` to your agent endpoint. The API is OpenAI-compatible. The agent access key is **not** interchangeable with your serverless model access key or a DigitalOcean API token.

## Chat with an agent

### cURL

Set `include_retrieval_info: true` to run RAG against the agent's attached knowledge bases (and to see which files were used).

```bash
curl -s -X POST "$AGENT_ENDPOINT/api/v1/chat/completions" \
  -H "Authorization: Bearer $AGENT_ACCESS_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "What can you help me with?"}],
    "stream": false,
    "include_retrieval_info": true
  }'
```

### Python — OpenAI SDK (recommended)

When using the OpenAI library, set `base_url` to `$AGENT_ENDPOINT/api/v1/` — do not append `/chat/completions` yourself.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(
    base_url=f"{os.getenv('AGENT_ENDPOINT')}/api/v1/",
    api_key=os.getenv("AGENT_ACCESS_KEY")
)

response = client.chat.completions.create(
    model="n/a",  # required by the SDK but ignored; the agent uses its configured model
    messages=[{"role": "user", "content": "What can you help me with?"}],
    extra_body={"include_retrieval_info": True}
)

print(response.choices[0].message.content)
```

## Knowledge bases

Attach a knowledge base to an agent for RAG. Create and manage knowledge bases under **INFERENCE > Agent Platform**, or via the control-plane API at `https://api.digitalocean.com/v2/gen-ai/knowledge_bases` (authenticated with a DigitalOcean API token). See [databases.md](databases.md) for the managed Knowledge Bases + OpenSearch workflow, and the docs below.

## Official docs

- [Use agents](https://docs.digitalocean.com/products/inference/how-to/use-agents/)
- [Create agents](https://docs.digitalocean.com/products/inference/how-to/create-agents/)
- [Attach agent knowledge bases](https://docs.digitalocean.com/products/inference/how-to/attach-agent-knowledge-bases/)
