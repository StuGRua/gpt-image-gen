# gpt-image-gen

A skill for generating and editing images using OpenAI GPT Image 2, compatible with Claude Code, Kiro CLI, Codex, and other AI coding agents.

## Installation

### Via npx skills (recommended)

```bash
npx skills add StuGRua/gpt-image-gen
```

### Manual installation

```bash
# Clone into Claude Code skills directory
git clone https://github.com/StuGRua/gpt-image-gen.git ~/.claude/skills/gpt-image-gen
```

## Configuration

Copy the example config and fill in your provider details:

```bash
cd ~/.claude/skills/gpt-image-gen   # or ~/.agents/skills/gpt-image-gen
cp local.config.example.json local.config.json
```

Edit `local.config.json`:

```json
{
  "base_url": "https://api.openai.com",
  "api_key": "sk-your-api-key-here",
  "model": "gpt-image-2",
  "default_size": "1024x1024",
  "default_quality": "medium",
  "default_format": "png",
  "unsupported": []
}
```

If your provider doesn't support certain features, add them to `unsupported`:

```json
"unsupported": ["background:transparent", "output_format:webp", "responses_api"]
```

## Usage

In Claude Code or compatible agents, trigger the skill with:

- `/gpt-image-gen` followed by your image description
- Or use natural language: "帮我画一张...", "generate an image of...", "生成图片"

## Features

- Bilingual prompt crafting (English + Simplified Chinese)
- Text-to-image generation via `/v1/images/generations`
- Image editing via `/v1/images/edits`
- Automatic base64 decoding and file output
- Provider-aware parameter handling (skips unsupported features)
- Iterative refinement workflow

## Model Reference

See [references/gpt-image-2.md](references/gpt-image-2.md) for detailed model capabilities, parameter constraints, and pricing.

## License

MIT
