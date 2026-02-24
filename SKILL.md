---
name: bedrock-image-gen
description: >-
  Generate and edit images via Amazon Bedrock using Nova Canvas and Stability AI
  (Stable Diffusion 3.5 Large, Stable Image Ultra, Stable Image Core) models.
  Use when the user wants to create images, generate artwork, edit photos,
  remove backgrounds, inpaint, outpaint, upscale, or do image-to-image
  transformations using AWS Bedrock. Covers text-to-image, image variation,
  color-guided generation, image conditioning, virtual try-on, and style transfer.
  CLI-only approach using aws cli, curl, base64, and jq.
context: fork
skills:
  - aws-mcp-setup
allowed-tools:
  - mcp__aws-mcp__*
  - mcp__awsdocs__*
  - Bash(aws bedrock-runtime *)
  - Bash(aws bedrock *)
  - Bash(aws sts get-caller-identity)
  - Bash(curl *bedrock*)
  - Bash(jq *)
  - Bash(base64 *)
hooks:
  PreToolUse:
    - matcher: Bash(aws bedrock-runtime invoke-model*)
      command: aws sts get-caller-identity --query Account --output text
      once: true
---

# Bedrock Image Generation Skill

Generate and edit images through **Amazon Bedrock** using **Amazon Nova Canvas** and **Stability AI** (Stable Diffusion 3.5 Large, Stable Image Ultra, Stable Image Core) models.

This skill uses **only CLI tools** `aws` CLI, `curl`, `base64`, `jq` no Python/SDK code.

---

## Prerequisites

1. **AWS CLI v2** installed and configured (`aws configure`)
2. **jq** installed (`apt-get install -y jq`)
3. **base64** (coreutils, pre-installed on Linux)
4. **curl** with `--aws-sigv4` support (curl 7.75+) for direct REST calls
5. AWS credentials with `bedrock:InvokeModel` permission
6. **Model access enabled** in the Bedrock console for the desired models

---

## Supported Models and Regional Availability

### Amazon Nova Canvas

| Model ID | Regions |
|---|---|
| `amazon.nova-canvas-v1:0` | `us-east-1`, `eu-west-1`, `ap-northeast-1` |

**Capabilities:** Text-to-image, image conditioning (Canny Edge / Segmentation), color-guided generation, image variation, inpainting, outpainting, background removal, virtual try-on.

### Stability AI Models

| Model | Model ID | Regions (single) | Cross-region |
|---|---|---|---|
| SD 3.5 Large | `stability.sd3-5-large-v1:0` | `us-west-2` | n/a |
| Stable Image Ultra 1.0 | `stability.stable-image-ultra-v1:1` | `us-west-2` | n/a |
| Stable Image Core 1.0 | `stability.stable-image-core-v1:1` | `us-west-2` | n/a |
| Stable Image Inpaint | `stability.stable-image-inpaint-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Image Outpaint | `stability.stable-outpaint-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Image Erase Object | `stability.stable-image-erase-object-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Image Remove BG | `stability.stable-image-remove-background-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Search and Replace | `stability.stable-image-search-replace-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Search and Recolor | `stability.stable-image-search-recolor-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Style Transfer | `stability.stable-style-transfer-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Control Sketch | `stability.stable-image-control-sketch-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Control Structure | `stability.stable-image-control-structure-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Style Guide | `stability.stable-image-style-guide-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Fast Upscale | `stability.stable-fast-upscale-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Conservative Upscale | `stability.stable-conservative-upscale-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |
| Stable Creative Upscale | `stability.stable-creative-upscale-v1:0` | n/a | `us-east-1`, `us-east-2`, `us-west-2` |

---

## API Endpoint Format

```
POST https://bedrock-runtime.{REGION}.amazonaws.com/model/{MODEL_ID}/invoke
Content-Type: application/json
Accept: application/json
```

## Method 1: AWS CLI (Recommended)

Handles SigV4 signing automatically.

### 1.1 Nova Canvas: Text-to-Image (basic)

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
PROMPT="A photorealistic golden retriever playing in autumn leaves"
OUTPUT_FILE="output.png"

jq -n --arg prompt "$PROMPT" '{
  taskType: "TEXT_IMAGE",
  textToImageParams: { text: $prompt },
  imageGenerationConfig: {
    width: 1024, height: 1024,
    quality: "standard", cfgScale: 6.5,
    seed: 42, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" \
  --model-id "$MODEL_ID" \
  --content-type "application/json" \
  --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json \
  /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > "$OUTPUT_FILE"
echo "Saved: $OUTPUT_FILE"
```

### 1.2 Nova Canvas: Text-to-Image with Style and Negative Prompt

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"

jq -n '{
  taskType: "TEXT_IMAGE",
  textToImageParams: {
    text: "A futuristic city skyline at sunset with flying cars",
    negativeText: "blurry, low quality, distorted, watermark",
    style: "PHOTOREALISM"
  },
  imageGenerationConfig: {
    width: 1280, height: 720,
    quality: "premium", cfgScale: 7.0,
    seed: 123, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > styled-output.png
```

**Available Nova Canvas styles:**
`3D_ANIMATED_FAMILY_FILM` / `DESIGN_SKETCH` / `FLAT_VECTOR_ILLUSTRATION` / `GRAPHIC_NOVEL_ILLUSTRATION` / `MAXIMALISM` / `MIDCENTURY_RETRO` / `PHOTOREALISM` / `SOFT_DIGITAL_PAINTING`

### 1.3 Nova Canvas: Image Conditioning (layout control)

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
CONDITION_B64=$(base64 -w 0 layout-reference.png)

jq -n --arg img "$CONDITION_B64" '{
  taskType: "TEXT_IMAGE",
  textToImageParams: {
    text: "A watercolor painting of a mountain village",
    conditionImage: $img,
    controlMode: "CANNY_EDGE",
    controlStrength: 0.7
  },
  imageGenerationConfig: {
    width: 1024, height: 1024,
    quality: "standard", cfgScale: 6.5,
    seed: 42, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > conditioned.png
```

**controlMode:** `CANNY_EDGE` (follows edges) / `SEGMENTATION` (follows layout shapes)

### 1.4 Nova Canvas: Color-Guided Generation

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"

jq -n '{
  taskType: "COLOR_GUIDED_GENERATION",
  colorGuidedGenerationParams: {
    text: "A serene landscape with rolling hills",
    colors: ["#2E8B57", "#87CEEB", "#FFD700", "#8B4513"]
  },
  imageGenerationConfig: {
    width: 1024, height: 1024,
    quality: "standard", cfgScale: 6.5,
    seed: 42, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > color-guided.png
```

### 1.5 Nova Canvas: Image Variation

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
SRC_B64=$(base64 -w 0 source.png)

jq -n --arg img "$SRC_B64" '{
  taskType: "IMAGE_VARIATION",
  imageVariationParams: {
    images: [$img],
    similarityStrength: 0.7,
    text: "Same scene but in winter with snow"
  },
  imageGenerationConfig: {
    width: 1024, height: 1024,
    cfgScale: 6.5, seed: 42, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > variation.png
```

### 1.6 Nova Canvas: Inpainting (edit region)

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
IMG_B64=$(base64 -w 0 input.png)

jq -n --arg img "$IMG_B64" '{
  taskType: "INPAINTING",
  inPaintingParams: {
    image: $img,
    maskPrompt: "the sky",
    text: "a dramatic stormy sky with lightning"
  },
  imageGenerationConfig: {
    numberOfImages: 1, quality: "standard",
    cfgScale: 6.5, seed: 42
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > inpainted.png
```

> **Tip:** Use `maskImage` (base64 B/W image) instead of `maskPrompt` for precise control. Black = change, white = keep.

### 1.7 Nova Canvas: Outpainting (replace background)

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
IMG_B64=$(base64 -w 0 input.png)

jq -n --arg img "$IMG_B64" '{
  taskType: "OUTPAINTING",
  outPaintingParams: {
    image: $img,
    maskPrompt: "the person",
    outPaintingMode: "PRECISE",
    text: "a beautiful tropical beach background"
  },
  imageGenerationConfig: {
    numberOfImages: 1, quality: "standard",
    cfgScale: 6.5, seed: 42
  }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > outpainted.png
```

**outPaintingMode:** `DEFAULT` (smooth transition, similar colors) / `PRECISE` (strict mask, better for big changes)

### 1.8 Nova Canvas: Background Removal

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
IMG_B64=$(base64 -w 0 input.png)

jq -n --arg img "$IMG_B64" '{
  taskType: "BACKGROUND_REMOVAL",
  backgroundRemovalParams: { image: $img }
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > no-background.png
```

> Returns PNG with full 8-bit transparency. No `imageGenerationConfig` needed.

### 1.9 Stable Diffusion 3.5 Large: Text-to-Image

```bash
REGION="us-west-2"
MODEL_ID="stability.sd3-5-large-v1:0"

jq -n '{
  prompt: "A majestic dragon perched on a crystal mountain, fantasy art",
  negative_prompt: "blurry, low quality",
  aspect_ratio: "16:9",
  seed: 42,
  output_format: "png"
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > sd35-output.png
```

**SD 3.5 Large aspect_ratio values:** `16:9` / `1:1` / `21:9` / `2:3` / `3:2` / `4:5` / `5:4` / `9:16` / `9:21`

### 1.10 Stable Diffusion 3.5 Large: Image-to-Image

```bash
REGION="us-west-2"
MODEL_ID="stability.sd3-5-large-v1:0"
INPUT_B64=$(base64 -w 0 input.jpg)

jq -n --arg img "$INPUT_B64" '{
  prompt: "Transform into an oil painting style",
  image: $img,
  mode: "image-to-image",
  strength: 0.7,
  seed: 42,
  output_format: "png"
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > sd35-img2img.png
```

### 1.11 Stable Image Ultra: Text-to-Image

```bash
REGION="us-west-2"
MODEL_ID="stability.stable-image-ultra-v1:1"

jq -n '{
  prompt: "A luxury watch on marble surface, product photography, studio lighting",
  negative_prompt: "blurry, low quality, cartoon",
  aspect_ratio: "1:1",
  seed: 42,
  output_format: "png"
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > ultra-output.png
```

### 1.12 Stable Image Core: Text-to-Image (fast and affordable)

```bash
REGION="us-west-2"
MODEL_ID="stability.stable-image-core-v1:1"

jq -n '{
  prompt: "A cute robot serving coffee in a cafe, digital art",
  aspect_ratio: "1:1",
  seed: 42,
  output_format: "png"
}' > /tmp/bedrock-req.json

aws bedrock-runtime invoke-model \
  --region "$REGION" --model-id "$MODEL_ID" \
  --content-type "application/json" --accept "application/json" \
  --body fileb:///tmp/bedrock-req.json /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > core-output.png
```

---

## Method 2: curl with AWS SigV4 Signing

Requires curl 7.75+ with `--aws-sigv4` support and exported AWS credentials.

### 2.1 Nova Canvas: Text-to-Image via curl

```bash
REGION="us-east-1"
MODEL_ID="amazon.nova-canvas-v1:0"
ENDPOINT="https://bedrock-runtime.${REGION}.amazonaws.com/model/${MODEL_ID}/invoke"

# Ensure AWS credentials are exported
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
# export AWS_SESSION_TOKEN="your-session-token"  # if using temporary creds

jq -n '{
  taskType: "TEXT_IMAGE",
  textToImageParams: {
    text: "A serene Japanese garden with cherry blossoms"
  },
  imageGenerationConfig: {
    width: 1024, height: 1024,
    quality: "standard", cfgScale: 6.5,
    seed: 42, numberOfImages: 1
  }
}' > /tmp/bedrock-req.json

curl -s -X POST "$ENDPOINT" \
  --aws-sigv4 "aws:amz:${REGION}:bedrock" \
  --user "${AWS_ACCESS_KEY_ID}:${AWS_SECRET_ACCESS_KEY}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d @/tmp/bedrock-req.json \
  -o /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > curl-nova.png
echo "Saved: curl-nova.png"
```

> **Note:** If using temporary credentials (STS/SSO), add header: `-H "x-amz-security-token: ${AWS_SESSION_TOKEN}"`

### 2.2 Stable Diffusion 3.5 Large via curl

```bash
REGION="us-west-2"
MODEL_ID="stability.sd3-5-large-v1:0"
ENDPOINT="https://bedrock-runtime.${REGION}.amazonaws.com/model/${MODEL_ID}/invoke"

jq -n '{
  prompt: "A cyberpunk cityscape at night with neon lights",
  aspect_ratio: "16:9",
  seed: 42,
  output_format: "png"
}' > /tmp/bedrock-req.json

curl -s -X POST "$ENDPOINT" \
  --aws-sigv4 "aws:amz:${REGION}:bedrock" \
  --user "${AWS_ACCESS_KEY_ID}:${AWS_SECRET_ACCESS_KEY}" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d @/tmp/bedrock-req.json \
  -o /tmp/bedrock-resp.json

jq -r '.images[0]' /tmp/bedrock-resp.json | base64 -d > curl-sd35.png
```

### 2.3 Inline one-liner via aws cli (pipe to file)

```bash
aws bedrock-runtime invoke-model \
  --region us-east-1 \
  --model-id amazon.nova-canvas-v1:0 \
  --content-type "application/json" \
  --accept "application/json" \
  --body '{"taskType":"TEXT_IMAGE","textToImageParams":{"text":"A cat astronaut"},"imageGenerationConfig":{"width":1024,"height":1024,"numberOfImages":1}}' \
  /dev/stdout | jq -r '.images[0]' | base64 -d > quick-output.png
```

---

## Nova Canvas imageGenerationConfig Reference

| Parameter | Type | Range | Default | Notes |
|---|---|---|---|---|
| `width` | int | 320-4096 (divisible by 16) | 1024 | Not for INPAINTING/OUTPAINTING/VIRTUAL_TRY_ON |
| `height` | int | 320-4096 (divisible by 16) | 1024 | Not for INPAINTING/OUTPAINTING/VIRTUAL_TRY_ON |
| `quality` | string | `standard` or `premium` | `standard` | Premium = higher detail, slower |
| `cfgScale` | float | 1.1-10.0 | 6.5 | Low(1-3)=creative, Mid(4-7)=balanced, High(8-10)=strict |
| `numberOfImages` | int | 1-5 | 1 | More images = longer generation time |
| `seed` | int | 0-2147483646 | 12 | Same seed + same params = same image |

**Resolution constraints:**
- Each side: 320-4096 pixels, divisible by 16
- Aspect ratio: between 1:4 and 4:1
- Total pixel count must be less than 4,194,304

## Nova Canvas Task Types Quick Reference

| Task | taskType | Key Params | Use Case |
|---|---|---|---|
| Text-to-Image | `TEXT_IMAGE` | `text`, `negativeText`, `style` | Generate from text prompt |
| Conditioned | `TEXT_IMAGE` | `text`, `conditionImage`, `controlMode` | Follow layout of reference image |
| Color-Guided | `COLOR_GUIDED_GENERATION` | `text`, `colors[]`, `referenceImage` | Match specific color palette |
| Variation | `IMAGE_VARIATION` | `images[]`, `similarityStrength`, `text` | Create variations of existing image |
| Inpainting | `INPAINTING` | `image`, `maskPrompt`/`maskImage`, `text` | Edit inside masked region |
| Outpainting | `OUTPAINTING` | `image`, `maskPrompt`/`maskImage`, `text` | Replace background outside mask |
| Background Removal | `BACKGROUND_REMOVAL` | `image` | Remove background (returns transparent PNG) |
| Virtual Try-On | `VIRTUAL_TRY_ON` | source image + reference image | Superimpose object onto source |

## Stability AI Response Format

All Stability AI models return:

```json
{
  "seeds": [2130420379],
  "finish_reasons": [null],
  "images": ["base64-encoded-image-data..."]
}
```

- `finish_reasons: [null]` = success
- `finish_reasons: ["Filter reason: prompt"]` = content filtered
- Other filter reasons: `"Filter reason: output image"`, `"Filter reason: input image"`, `"Inference error"`

## Nova Canvas Response Format

```json
{
  "images": ["base64-encoded-image-data..."],
  "error": "optional error message if RAI policy triggered"
}
```

---

## Troubleshooting

| Issue | Solution |
|---|---|
| `AccessDeniedException` | Enable model access in Bedrock console, check IAM permissions |
| `ResourceNotFoundException` | Wrong model ID or region, check availability table above |
| `ValidationException` | Check JSON payload format, resolution constraints |
| `ThrottlingException` | Rate limit hit, retry with backoff |
| `ModelTimeoutException` | Increase timeout, reduce quality/resolution/numberOfImages |
| curl `--aws-sigv4` not recognized | Upgrade curl to 7.75+ |
| Empty or corrupt image | Check `jq -r '.images[0]'` output is valid base64, check `finish_reasons` |

## Tips

- **Prompt quality matters:** Be specific and descriptive. Use `negativeText`/`negative_prompt` to exclude unwanted elements.
- **Avoid negation in prompts:** Do not use "no", "not", "without" in text prompts. Put unwanted elements in `negativeText` instead.
- **Seed reproducibility:** Same seed + same parameters = same image. Great for iterating on prompts.
- **Timeout:** For premium quality or multiple images, increase AWS CLI timeout: `aws configure set cli_read_timeout 300`
- **Large payloads:** When sending base64 images, the JSON can be very large. Always use `fileb://` for the body, not inline JSON.
- **Cost:** Nova Canvas and Stability AI models are billed per image generated. Check current pricing at https://aws.amazon.com/bedrock/pricing/

---

## Documentation Links

- [Amazon Bedrock InvokeModel API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html)
- [Nova Canvas Image Generation](https://docs.aws.amazon.com/nova/latest/userguide/image-gen-access.html)
- [Nova Canvas Request/Response Structure](https://docs.aws.amazon.com/nova/latest/userguide/image-gen-req-resp-structure.html)
- [Supported Bedrock Models](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)
- [Stability AI Models on Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-stability-diffusion.html)
- [SD 3.5 Large Parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-diffusion-3-5-large.html)
- [Stable Image Ultra Parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-diffusion-stable-ultra-text-image-request-response.html)
