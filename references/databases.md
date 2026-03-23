# Databases

DigitalOcean Managed Databases for AI-powered apps. Provision via the [DO Console](https://cloud.digitalocean.com) (recommended for quickest setup) or `doctl` CLI if preferred.

## PostgreSQL + pgvector

Best for: relational data + vector embeddings (RAG, similarity search, recommendations).

### Provision

Create a PostgreSQL cluster in the [DO Console](https://cloud.digitalocean.com) under Databases > Create Database Cluster > PostgreSQL.

Alternatively, via CLI:

```bash
doctl databases create my-ai-db --engine pg --version 16 --region nyc3 --size db-s-1vcpu-1gb --num-nodes 1 --wait
```

### Connection

Get the connection string from the Console (Databases > your cluster > Connection Details) or CLI:

```bash
doctl databases connection <cluster-id>
```

Connection string format:

```
postgresql://<username>:<password>@<hostname>:<port>/<database>?sslmode=require
```

SSL is required. Download the CA certificate from the cluster overview page.

### pgvector setup

```sql
CREATE EXTENSION vector;

CREATE TABLE embeddings (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1536)
);

CREATE INDEX ON embeddings USING hnsw (embedding vector_cosine_ops);
```

### Store embeddings from Gradient

```python
import os
import psycopg2
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

gradient = OpenAI(
    base_url="https://inference.do-ai.run/v1/",
    api_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY")
)

conn = psycopg2.connect(os.getenv("DATABASE_URL"))
cur = conn.cursor()

# Generate embedding (use a model that returns embeddings)
# Then store it
text = "Your content here"
cur.execute(
    "INSERT INTO embeddings (content, embedding) VALUES (%s, %s)",
    (text, embedding_vector)
)
conn.commit()

# Query by similarity
cur.execute("""
    SELECT content, 1 - (embedding <=> %s::vector) AS similarity
    FROM embeddings
    ORDER BY embedding <=> %s::vector
    LIMIT 5
""", (query_vector, query_vector))
results = cur.fetchall()
```

## MongoDB

Best for: flexible document storage — chat history, user profiles, AI response logs, unstructured data.

### Provision

Create a MongoDB cluster in the [DO Console](https://cloud.digitalocean.com) under Databases > Create Database Cluster > MongoDB.

Alternatively, via CLI:

```bash
doctl databases create my-mongo-db --engine mongodb --version 7 --region nyc3 --size db-s-1vcpu-1gb --num-nodes 1 --wait
```

### Connection

Get the connection string from the Console (Databases > your cluster > Connection Details).

Connection string format:

```
mongodb+srv://<username>:<password>@<hostname>/<database>?tls=true&authSource=admin
```

### Python example

```python
import os
from pymongo import MongoClient
from dotenv import load_dotenv

load_dotenv()

client = MongoClient(os.getenv("DATABASE_URL"))
db = client["myapp"]

# Store chat history
db.conversations.insert_one({
    "user_id": "user123",
    "messages": [
        {"role": "user", "content": "Hello!"},
        {"role": "assistant", "content": "Hi there!"}
    ],
    "model": "llama3.3-70b-instruct",
    "created_at": datetime.utcnow()
})

# Query conversations
history = db.conversations.find({"user_id": "user123"}).sort("created_at", -1).limit(10)
```

### Node.js example

```javascript
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.DATABASE_URL);
const db = client.db("myapp");

// Store chat history
await db.collection("conversations").insertOne({
  userId: "user123",
  messages: [
    { role: "user", content: "Hello!" },
    { role: "assistant", content: "Hi there!" },
  ],
  model: "llama3.3-70b-instruct",
  createdAt: new Date(),
});
```

## Gradient Knowledge Bases + OpenSearch

Best for: fully managed vector search — no database setup required. Gradient handles chunking, embedding, and indexing automatically.

### How it works

1. Create a knowledge base in the [DO Console](https://cloud.digitalocean.com) under Gradient > Agents > Knowledge Bases
2. Select an embedding model
3. Connect a data source (file upload, DO Spaces, public URL, S3, or Dropbox)
4. Gradient chunks the data, generates embeddings, and stores them in a managed OpenSearch cluster

The OpenSearch cluster appears in your Databases list and scales independently.

### Available embedding models

| Model | Token Window | Chunk Size |
|---|---|---|
| GTE Large (v1.5) | 8,192 | 0-750 chars |
| All-MiniLM-L6-v2 | 256 | 0-256 chars |
| Multi-QA-mpnet-base-dot-v1 | 512 | 0-512 chars |
| Qwen3 Embedding 0.6B | 8,000 | 0-750 chars |

### Chunking strategies

- **Section-based** (default) — uses structural elements like headings
- **Semantic** — groups by meaning
- **Hierarchical** — parent/child relationships
- **Fixed-length** — strict token count

### Query via agents

Once a knowledge base is attached to an agent, query it using the [Agents API](agents.md). The agent automatically retrieves relevant context from OpenSearch.

## Connecting databases to App Platform

When deploying to App Platform, bind your database to the app for automatic env var injection.

### App spec with database

```yaml
name: my-gradient-app
databases:
  - name: my-db
    engine: PG
    production: false
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
      - key: DATABASE_URL
        scope: RUN_TIME
        value: ${my-db.DATABASE_URL}
```

Setting `production: false` creates a dev database (free tier). Set to `true` for production clusters.

### Auto-injected variables

When a database is bound, these variables are available:

| Variable | Description |
|---|---|
| `${db.DATABASE_URL}` | Full connection string |
| `${db.HOSTNAME}` | Database host |
| `${db.PORT}` | Database port |
| `${db.USERNAME}` | Database user |
| `${db.PASSWORD}` | Database password |
| `${db.DATABASE}` | Database name |
| `${db.CA_CERT}` | TLS CA certificate |

Replace `db` with your database name from the app spec.

## Official docs

- [Managed Databases overview](https://docs.digitalocean.com/products/databases/)
- [PostgreSQL](https://docs.digitalocean.com/products/databases/postgresql/)
- [MongoDB](https://docs.digitalocean.com/products/databases/mongodb/)
- [App Platform database management](https://docs.digitalocean.com/products/app-platform/how-to/manage-databases/)
