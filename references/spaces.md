# DigitalOcean Spaces

S3-compatible object storage for storing generated images, audio files, uploads, and static assets. Includes built-in CDN.

## Setup

1. Create a Spaces bucket in the [DO Console](https://cloud.digitalocean.com) under Spaces Object Storage > Create Bucket
2. Generate access keys: Spaces Object Storage > Spaces Access Keys > Create Access Key
3. Set environment variables:

```bash
export SPACES_KEY="your-access-key-id"
export SPACES_SECRET="your-secret-access-key"
```

The secret key is shown only once at creation — copy it immediately.

## Endpoint

```
https://{region}.digitaloceanspaces.com
```

Example: `https://nyc3.digitaloceanspaces.com`

Available regions: `nyc3`, `sfo3`, `ams3`, `sgp1`, `fra1`, `lon1`, `tor1`, `blr1`, `syd1`, and more.

## CDN endpoint

```
https://{bucket-name}.{region}.cdn.digitaloceanspaces.com
```

Enable CDN in the Console under your bucket's Settings. Use CDN URLs for serving images and assets to users.

## Python (boto3)

```bash
pip install boto3
```

```python
import os
import boto3
from botocore.client import Config

session = boto3.session.Session()
client = session.client(
    "s3",
    region_name="nyc3",
    endpoint_url="https://nyc3.digitaloceanspaces.com",
    aws_access_key_id=os.getenv("SPACES_KEY"),
    aws_secret_access_key=os.getenv("SPACES_SECRET"),
    config=Config(s3={"addressing_style": "virtual"})
)

# Upload a file
client.put_object(
    Bucket="my-bucket",
    Key="images/generated.png",
    Body=open("generated.png", "rb"),
    ACL="public-read"  # or "private"
)

# Upload bytes directly (e.g., from base64-decoded image)
import base64
image_bytes = base64.b64decode(b64_image_data)
client.put_object(
    Bucket="my-bucket",
    Key="images/ai-generated.png",
    Body=image_bytes,
    ContentType="image/png",
    ACL="public-read"
)

# Download a file
client.download_file("my-bucket", "images/generated.png", "local-copy.png")

# List files
response = client.list_objects_v2(Bucket="my-bucket", Prefix="images/")
for obj in response.get("Contents", []):
    print(obj["Key"], obj["Size"])
```

## Node.js (@aws-sdk/client-s3)

```bash
npm install @aws-sdk/client-s3
```

```javascript
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import fs from "fs";

const client = new S3Client({
  endpoint: "https://nyc3.digitaloceanspaces.com",
  region: "us-east-1",
  forcePathStyle: false,
  credentials: {
    accessKeyId: process.env.SPACES_KEY,
    secretAccessKey: process.env.SPACES_SECRET,
  },
});

// Upload a file
await client.send(
  new PutObjectCommand({
    Bucket: "my-bucket",
    Key: "images/generated.png",
    Body: fs.createReadStream("generated.png"),
    ContentType: "image/png",
    ACL: "public-read",
  })
);

// Upload a buffer (e.g., from base64-decoded image)
const imageBuffer = Buffer.from(b64ImageData, "base64");
await client.send(
  new PutObjectCommand({
    Bucket: "my-bucket",
    Key: "images/ai-generated.png",
    Body: imageBuffer,
    ContentType: "image/png",
    ACL: "public-read",
  })
);
```

Note: Use `us-east-1` as the region value even if your bucket is in a different region.

## Common pattern: Save Inference-generated images to Spaces

```python
import os
import base64
import boto3
from botocore.client import Config
from openai import OpenAI

inference = OpenAI(
    base_url="https://inference.do-ai.run/v1/",
    api_key=os.getenv("DIGITAL_OCEAN_MODEL_ACCESS_KEY")
)

spaces = boto3.session.Session().client(
    "s3",
    region_name="nyc3",
    endpoint_url="https://nyc3.digitaloceanspaces.com",
    aws_access_key_id=os.getenv("SPACES_KEY"),
    aws_secret_access_key=os.getenv("SPACES_SECRET"),
    config=Config(s3={"addressing_style": "virtual"})
)

# Generate image with the Inference Engine
response = inference.images.generate(
    model="openai-gpt-image-2",
    prompt="A futuristic cityscape at sunset",
    n=1,
    size="1024x1024"
)

# Upload to Spaces
image_bytes = base64.b64decode(response.data[0].b64_json)
spaces.put_object(
    Bucket="my-bucket",
    Key="generated/cityscape.png",
    Body=image_bytes,
    ContentType="image/png",
    ACL="public-read"
)

# Public URL
url = "https://my-bucket.nyc3.cdn.digitaloceanspaces.com/generated/cityscape.png"
```

## Public vs private access

- Files are **private by default**
- Set `ACL="public-read"` on upload to make files publicly accessible
- For time-limited access to private files, use presigned URLs:

```python
url = client.generate_presigned_url(
    "get_object",
    Params={"Bucket": "my-bucket", "Key": "private/file.png"},
    ExpiresIn=3600  # seconds
)
```

## App Platform environment variables

Add these to your app spec:

```yaml
envs:
  - key: SPACES_KEY
    scope: RUN_TIME
    type: SECRET
  - key: SPACES_SECRET
    scope: RUN_TIME
    type: SECRET
```

## Official docs

- [Spaces overview](https://docs.digitalocean.com/products/spaces/)
- [S3 SDK examples](https://docs.digitalocean.com/products/spaces/how-to/use-aws-sdks/)
- [CDN](https://docs.digitalocean.com/products/spaces/how-to/enable-cdn/)
