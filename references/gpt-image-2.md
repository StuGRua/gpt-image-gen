# GPT Image 2 Reference

## Model Facts

- Model ID: `gpt-image-2`
- Released: April 21, 2026
- Architecture: Standalone single-pass inference (not built on GPT-4o pipeline)
- Text rendering accuracy: ~99%
- Max resolution: 2K standard, 4K beta via API
- Knowledge cutoff: December 2025
- Modes: Instant (default) and Thinking (reasoning + web search, ChatGPT only)

## API Endpoints

### Images API (recommended for most use cases)

| Endpoint | Method | Content-Type | Use Case |
| --- | --- | --- | --- |
| `/v1/images/generations` | POST | `application/json` | Text-to-image |
| `/v1/images/edits` | POST | `multipart/form-data` | Image editing with source images |

### Responses API (for multi-step workflows)

| Endpoint | Method | Note |
| --- | --- | --- |
| `/v1/responses` | POST | Top-level `model` must be a text model (e.g. `gpt-4.1-mini`), image gen is a tool |

**Important**: Most third-party providers only support the Images API. The Responses API and Chat Completions API are generally not available for image generation through proxies.

## Parameters — `/v1/images/generations`

| Parameter | Type | Required | Values | Default |
| --- | --- | --- | --- | --- |
| `model` | string | Yes | `gpt-image-2` (fixed, never use older models) | `gpt-image-2` |
| `prompt` | string | Yes | Max 32,000 chars | — |
| `size` | string | No | `1024x1024`, `1024x1536`, `1536x1024`, `auto` | `1024x1024` |
| `quality` | string | No | `low`, `medium`, `high`, `auto` | `auto` |
| `n` | integer | No | 1–10 | 1 |
| `output_format` | string | No | `png`, `jpeg`, `webp` | `png` |
| `background` | string | No | `transparent`, `opaque`, `auto` | `auto` |

## Parameters — `/v1/images/edits`

Same as generations, plus:

| Parameter | Type | Required | Note |
| --- | --- | --- | --- |
| `image[]` | file | Yes | Multipart file upload, supports multiple images |
| `input_fidelity` | string | No | `low`, `high` — controls preservation of source image details |

## Response Format

GPT Image models return `b64_json` by default. URL output is **not supported**.

```json
{
  "data": [
    {
      "b64_json": "<base64-encoded-image-data>",
      "revised_prompt": "<model's interpretation of the prompt>"
    }
  ],
  "usage": {
    "input_tokens": 20,
    "output_tokens": 805,
    "total_tokens": 825
  }
}
```

## Pricing (per 1M tokens)

| Modality | Input | Cached Input | Output |
| --- | --- | --- | --- |
| Image | $8.00 | $2.00 | $30.00 |
| Text | $5.00 | $1.25 | $10.00 |

Per-image cost: ~$0.006 (low/small) to ~$0.211 (high/large).

## Provider-Specific Notes (yunwu.ai)

Tested on 2026-04-22:

| Feature | Status |
| --- | --- |
| `/v1/images/generations` | Supported |
| `/v1/images/edits` | Not tested |
| `/v1/responses` | Not supported (429) |
| `/v1/chat/completions` | Not supported |
| `size: 1024x1024` | Supported |
| `size: 1536x1024` | Supported |
| `quality: medium` | Supported |
| `quality: high` | Supported (slow, ~2min) |
| `output_format: png` | Supported |
| `output_format: jpeg` | Supported |
| `output_format: webp` | Not supported |
| `background: transparent` | Not supported |

## Sources

- [OpenAI Official Community Post](https://community.openai.com/t/introducing-gpt-image-2-available-today-in-the-api-and-codex/1379479)
- [OpenAI Image Generation Guide](https://platform.openai.com/docs/guides/image-generation)
- [OpenAI Images API Reference](https://platform.openai.com/docs/api-reference/images/create)
- [HandyAI: Model Drop GPT Image 2](https://handyai.substack.com/p/model-drop-gpt-image-2)
