---
name: gpt-image-gen
version: 1.0.0
description: >-
  Use when the user wants to generate or edit images using OpenAI GPT Image 2.
  Handles prompt refinement, bilingual prompt delivery, parameter selection,
  API call via Images API, and result decoding.
  Triggers on keywords: 生图, 画图, 图片生成, image generation, generate image,
  create image, 生成图片, 做张图, gpt-image, 图片编辑, image edit, draw, 画一张,
  make an image, create a picture, 做个图, 帮我画, 生成一张图.
  IMPORTANT: This skill ONLY uses gpt-image-2. Never fall back to gpt-image-1.5,
  gpt-image-1, gpt-image-1-mini, or any DALL-E model.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# GPT Image Gen

## Overview

Turn a rough image idea into a generation-ready English prompt plus a faithful Simplified Chinese translation, confirm with the user, then call the OpenAI Images API to generate or edit images. Decode the base64 result and present the image.

## Workflow

### Step 1: Establish the Task

- If the user did not provide the image idea, ask for it.
- Distinguish between **text-to-image** (`/v1/images/generations`) and **image editing** (`/v1/images/edits`).
- If the user provides reference images, switch to the edit endpoint.

### Step 2: Preflight Questions

Ask only what's missing. Skip questions the user already answered.

- **quality**: `low`, `medium`, `high` (default: `medium` for drafts, `high` for final)
- **size**: `1024x1024`, `1024x1536`, `1536x1024`, `auto` (default: `1024x1024`)
- **output format**: `png`, `jpeg` (default: `png`)
- **count** (`n`): 1–10 (default: 1)
- **reference images**: none, or each image plus its editing role

Quick-resolve ambiguous wording:

| User wording | Interpret as |
| --- | --- |
| 高清 / high quality | `quality: high`, `size: 1536x1024` or `1024x1536` |
| 快速草稿 / quick draft | `quality: low`, `size: 1024x1024` |
| 竖版 / portrait / 手机壁纸 | `size: 1024x1536` |
| 横版 / landscape / 封面 | `size: 1536x1024` |
| 正方形 / square | `size: 1024x1024` |

### Step 3: Shape the Prompt

Build one English prompt as the generation source of truth, then produce one Simplified Chinese translation.

Include only fields that materially improve output:

- subject
- scene or background
- style or medium
- composition or framing
- lighting or mood
- materials or textures
- text to render (if any — gpt-image-2 excels at text rendering)
- constraints and avoid list
- reference image roles (for edits)

Do not pad with generic adjectives. Add detail only when it changes the likely result.

### Step 4: Confirm Before Generating

Present the prompt in this format:

**English Prompt**
```text
<final English prompt>
```

**中文翻译**
```text
<faithful Simplified Chinese translation>
```

**Generation Settings**
- Model: `<model>`
- Size: `<size>`
- Quality: `<quality>`
- Format: `<output_format>`
- Count: `<n>`
- Endpoint: `<generation or edit>`

Wait for explicit user confirmation. Do not call any API before confirmation.

### Step 5: Load Configuration

Read `local.config.json` in the skill root for provider settings.

File path: `local.config.json` in the skill directory.

If the file exists, read `base_url`, `api_key`, `model`, and any `unsupported` list.
If the file does not exist, ask the user for provider details and create it.

Do not fall back to `OPENAI_API_KEY` or `OPENAI_BASE_URL`. Image generation must use its own explicit config.

### Step 6: Call the API

**Text-to-Image** — `POST {base_url}/v1/images/generations`

```bash
curl -s "{base_url}/v1/images/generations" \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "{english_prompt}",
    "size": "{size}",
    "quality": "{quality}",
    "output_format": "{output_format}",
    "n": {n}
  }' > response.json
```

**Image Editing** — `POST {base_url}/v1/images/edits` (multipart)

```bash
curl -s -X POST "{base_url}/v1/images/edits" \
  -H "Authorization: Bearer {api_key}" \
  -F "model=gpt-image-2" \
  -F "image[]=@{source_image}" \
  -F "prompt={english_prompt}" \
  -F "quality={quality}" > edit-response.json
```

**Python SDK** (preferred for complex workflows):

```python
from openai import OpenAI
import base64

client = OpenAI(api_key="{api_key}", base_url="{base_url}")

# Text-to-Image
response = client.images.generate(
    model="gpt-image-2",
    prompt="{english_prompt}",
    size="{size}",
    quality="{quality}",
    n={n},
)

for i, img in enumerate(response.data):
    with open(f"output_{i}.png", "wb") as f:
        f.write(base64.b64decode(img.b64_json))
```

### Step 7: Decode and Present

GPT Image models return `b64_json` by default. URL output is not supported.

```bash
# macOS
jq -r '.data[0].b64_json' response.json | base64 -D > output.png

# Linux
jq -r '.data[0].b64_json' response.json | base64 --decode > output.png
```

After decoding:
1. Verify the file with `file output.png` to confirm valid image data.
2. Present the image to the user using the Read tool.
3. Report token usage from `response.usage` if available.

### Step 8: Iterate

If the user wants changes:
- For prompt tweaks: update the English prompt, re-confirm, re-generate.
- For editing existing output: switch to the edit endpoint with the generated image as input.
- Preserve conversation context across iterations.

## Configuration Contract

Use `local.config.json` in the skill root. Do not reuse generic `OPENAI_API_KEY` or `OPENAI_BASE_URL`.

```json
{
  "base_url": "https://yunwu.ai",
  "api_key": "<secret>",
  "model": "gpt-image-2",
  "default_size": "1024x1024",
  "default_quality": "medium",
  "default_format": "png"
}
```

If the local config file does not exist, ask the user for:
1. Provider base URL
2. API key
3. Model name

Then create the config file.

## Model Reference

See [references/gpt-image-2.md](references/gpt-image-2.md) for model capabilities, parameter constraints, and provider-specific limitations.

## Provider Limitations

Different providers may not support all official OpenAI parameters. Known restrictions should be documented in `local.config.json` under `"unsupported"`:

```json
{
  "base_url": "https://yunwu.ai",
  "api_key": "<secret>",
  "model": "gpt-image-2",
  "unsupported": ["background:transparent", "output_format:webp", "responses_api"]
}
```

When a parameter is unsupported, silently skip it rather than sending it and getting an error.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Calling API before user confirms prompt | Always wait for explicit confirmation |
| Returning only one language version | Always provide both English and Chinese |
| Sending `background: transparent` without checking provider support | Check `unsupported` list in config first |
| Using `/v1/responses` or `/v1/chat/completions` for image gen | Use `/v1/images/generations` — it's the only reliable path for most providers |
| Trying to fetch `.data[0].url` | GPT Image models return `b64_json` only |
| Sending `output_format: webp` without checking | Some providers only support `png` and `jpeg` |
| Reusing generic OpenAI client config | Use dedicated image-gen config from `local.config.json` |
| Falling back to gpt-image-1.5, gpt-image-1, or DALL-E | This skill only uses `gpt-image-2`. If the API errors, report the error — do not downgrade the model |
