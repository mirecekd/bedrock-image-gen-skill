# Bedrock Image Generation Skill

AI agent skill for generating and editing images via **Amazon Bedrock** using **Nova Canvas** and **Stability AI** (Stable Diffusion 3.5 Large, Stable Image Ultra, Stable Image Core) models.

Compatible with [Agent Zero](https://github.com/frdel/agent-zero) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Features

- **Text-to-Image** generation with style presets and negative prompts
- **Image-to-Image** transformation and style transfer
- **Image Variation** — create variations of existing images
- **Inpainting** — edit specific regions using natural language masks
- **Outpainting** — replace or extend backgrounds
- **Background Removal** — transparent PNG output
- **Color-Guided Generation** — match specific color palettes
- **Image Conditioning** — follow layout of reference images (Canny Edge / Segmentation)
- **Virtual Try-On** — superimpose objects onto source images

## Supported Models

| Model | Model ID | Regions |
|---|---|---|
| Amazon Nova Canvas | `amazon.nova-canvas-v1:0` | `us-east-1`, `eu-west-1`, `ap-northeast-1` |
| SD 3.5 Large | `stability.sd3-5-large-v1:0` | `us-west-2` |
| Stable Image Ultra | `stability.stable-image-ultra-v1:1` | `us-west-2` |
| Stable Image Core | `stability.stable-image-core-v1:1` | `us-west-2` |
| + 13 more Stability AI services | (inpaint, outpaint, upscale, erase, style transfer...) | cross-region |

## Prerequisites

- **AWS CLI v2** installed and configured (`aws configure`)
- **jq** installed (`apt-get install -y jq`)
- **base64** (coreutils, pre-installed on Linux)
- AWS credentials with `bedrock:InvokeModel` permission
- **Model access enabled** in the [Bedrock console](https://console.aws.amazon.com/bedrock/home#/modelaccess)

## Installation

### Agent Zero

Copy the `SKILL.md` file to your Agent Zero skills directory:

```bash
mkdir -p /path/to/agent-zero/skills/bedrock-image-gen
cp SKILL.md /path/to/agent-zero/skills/bedrock-image-gen/
```

The skill will be automatically detected and available in Agent Zero's system prompt.

### Claude Code

Copy the `SKILL.md` file to your Claude Code skills directory:

```bash
mkdir -p .claude/skills/bedrock-image-gen
cp SKILL.md .claude/skills/bedrock-image-gen/
```

The skill uses Claude Code 2.1 frontmatter features:
- `context: fork` — runs in isolated sub-agent context
- `skills: [aws-mcp-setup]` — auto-loads AWS MCP setup dependency
- `allowed-tools` — whitelisted bash commands and MCP tools
- `hooks` — verifies AWS identity before first `invoke-model` call

## Quick Start

Once installed, just ask your AI agent naturally:

- *"Generate an image of a golden retriever playing in autumn leaves"*
- *"Create a futuristic city at sunset via Bedrock"*
- *"Remove the background from /tmp/photo.png"*
- *"Make a variation of this image but in winter"*

The agent will load the skill, select the appropriate task type, build the JSON payload, and execute the AWS CLI command.

## CLI-Only Approach

This skill uses **no Python SDK or code** — only CLI tools:

- `aws bedrock-runtime invoke-model` — primary method (recommended)
- `curl --aws-sigv4` — alternative REST method
- `jq` — JSON payload construction and response parsing
- `base64` — image encoding/decoding

## What's Inside SKILL.md

- 12 complete command examples (8 Nova Canvas + 4 Stability AI)
- 3 curl/REST examples with SigV4 signing
- Full model and regional availability tables
- `imageGenerationConfig` parameter reference
- Task types quick reference table
- Response format documentation
- Troubleshooting guide
- Tips and best practices
- Links to official AWS documentation

## Related Projects

- [novareel-mcp](https://github.com/mirecekd/novareel-mcp) — MCP server for video generation with Amazon Nova Reel
- [awsblogs-mcp](https://github.com/mirecekd/awsblogs-mcp) — MCP server for AWS blog search
- [aws-skills](https://github.com/zxkane/aws-skills) — AWS skills plugin marketplace for Claude Code

## License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center"><a href="https://www.buymeacoffee.com/mirecekdg"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png" alt="Buy Me A Coffee"></a></div>
