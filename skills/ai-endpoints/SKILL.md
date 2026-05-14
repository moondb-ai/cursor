---
name: ai-endpoints
description: Define and call AI endpoints (text generation, image generation) in MoonDB. Use when adding AI features to an app.
---

# AI Endpoints

## When to use

- Adding AI-powered features (summarization, generation, classification) to an app
- Generating images from prompts
- Integrating LLM or image model capabilities without managing infrastructure

## Define endpoints in schema

Add `ai_endpoints` at the top level of your schema:

```json
{
  "tables": { ... },
  "ai_endpoints": {
    "summarize": {
      "model": "gemma",
      "prompt": "Summarize this text in 2 sentences: {{text}}",
      "access": "auth"
    },
    "generate_title": {
      "model": "gpt-oss",
      "prompt": "Generate a catchy blog title about: {{topic}}",
      "access": "auth"
    },
    "cover_image": {
      "model": "flux-schnell",
      "prompt": "A blog cover image about {{topic}}, modern minimal style",
      "access": "admin"
    }
  }
}
```

## Available models

### Text models

| Model   | Speed | Quality       | Cost         |
|---------|-------|---------------|--------------|
| gemma   | Fast  | Good          | 1-2 credits/1K tokens |
| gpt-oss | Slower| Most intelligent | 2-4 credits/1K tokens |

### Image models

| Model        | Speed | Quality         | Cost             |
|--------------|-------|-----------------|------------------|
| flux-schnell | Fast  | Good            | 10 credits/image |
| flux-dev     | Slower| Photorealistic  | 100 credits/image|

## Call an endpoint

```
POST /p/{project_id}/ai/{endpoint_name}
Authorization: Bearer {token}
Content-Type: application/json

{ "text": "Long article content here..." }
```

### Text response

```json
{
  "data": {
    "result": "This article discusses...",
    "model": "gemma",
    "credits_used": 3
  }
}
```

### Image response

```json
{
  "data": {
    "image_base64": "<base64-encoded-data>",
    "content_type": "image/png",
    "model": "flux-schnell",
    "credits_used": 10
  }
}
```

Image models return base64-encoded data, NOT a URL. Decode and display or upload to storage.

## Credits

Each plan includes free monthly credits:
- New Moon (free): 5,000 credits/mo
- Half Moon ($9): 25,000 credits/mo
- Full Moon ($29): 100,000 credits/mo
- Eclipse ($99): 500,000 credits/mo

Purchase more:
```
POST /v1/ai/topup
X-API-Key: mk_...
Content-Type: application/json

{ "package": "starter" }
```

Packages: starter ($5 / 10K credits), pro ($20 / 50K credits), business ($50 / 150K credits).

## Access levels

- `public` — anyone can call
- `auth` (default) — requires Bearer token
- `admin` — requires admin key only
