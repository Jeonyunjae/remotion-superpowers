---
name: generate-image
description: Generate AI images for your Remotion project. Describe what you need — backgrounds, thumbnails, characters, abstract art — and get production-ready images via your configured provider (free or paid).
---

# Generate Image — AI Image Creation

You are helping the user generate custom images for their Remotion video project.

**Load the `remotion-production` skill** for the `image-generation` and `model-providers` rules.

## Workflow

### 1. Understand the Need

Ask the user (if not already clear):
- **What is the image for?** — Scene background, thumbnail, character, product shot, abstract art, diagram?
- **Style** — Photorealistic, illustration, 3D render, flat design, watercolor, pixel art?
- **Dimensions** — Landscape (1920x1080), portrait (1080x1920), square (1080x1080)?
- **Mood/Colors** — Bright, dark, warm, cool, neon, pastel?

### 2. Check Provider Configuration

Read `config.yaml` to determine the image provider:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, ask the user: "Would you like to run `/select-models` to choose your image provider, or use the default (pixazo, free)?"

### 3. Craft the Prompt

Write a detailed image generation prompt. Good prompts include:
- Subject description
- Art style
- Lighting and mood
- Camera angle/perspective
- Color palette
- Background details

**Good prompt examples:**
- "Modern tech startup office, aerial view, clean minimalist design, bright natural lighting, white and blue color scheme, photorealistic"
- "Abstract geometric pattern, dark background, glowing neon purple and blue lines, futuristic, 3D render"
- "Friendly robot mascot waving, cartoon illustration style, pastel colors, clean white background, suitable for logo"
- "Golden hour cityscape, drone aerial shot, warm orange and pink sky, downtown skyline silhouette, cinematic"

### 4. Generate Image

Use the configured provider.

#### pixazo provider (FREE):

```bash
curl -s -X POST "https://api.pixazo.com/v1/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[crafted prompt]",
    "model": "flux-schnell",
    "width": 1920,
    "height": 1080,
    "num_inference_steps": 4
  }' | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
img_data = base64.b64decode(data['images'][0])
with open('public/images/[descriptive-name].png', 'wb') as f:
    f.write(img_data)
print('Image saved to public/images/[descriptive-name].png')
"
```

**Models available:**
- `flux-schnell` — Fast, 4 steps, good quality (default)
- `flux-dev` — Slower, 20 steps, better quality

#### flux-local provider (FREE, requires ComfyUI):

Requires ComfyUI running locally on port 8188. Use the ComfyUI API to queue a generation workflow:

```bash
# Verify ComfyUI is running
curl -s http://127.0.0.1:8188/system_stats > /dev/null && echo "ComfyUI: running" || echo "ComfyUI: not running — start with: cd /path/to/ComfyUI && python main.py"
```

Then queue a FLUX generation workflow via the ComfyUI API. See the `model-providers` rule for the full API pattern.

#### replicate provider (PAID):

```
# FLUX for high-quality images
replicate_run:
  model: "black-forest-labs/flux-1.1-pro"
  input:
    prompt: "[crafted prompt]"
    aspect_ratio: "16:9"
    output_format: "png"
```

Download the output to the project:
```bash
curl -o public/images/[descriptive-name].png "[replicate_output_url]"
```

Available Replicate models:
- **FLUX 1.1 Pro** (`black-forest-labs/flux-1.1-pro`) — High quality, great prompt following
- **Imagen 4** (`google/imagen-4`) — Google's latest, photorealistic
- **Ideogram v3** (`ideogram-ai/ideogram-v3`) — Best for text-in-image
- **FLUX Kontext** (`black-forest-labs/flux-kontext-pro`) — Style control, multi-reference

See `skills/remotion-production/rules/replicate-models.md` and `rules/image-generation.md` for detailed prompt engineering and model selection guidance.

#### kie provider (PAID):

```
Use remotion-media generate_image:
- prompt: [crafted prompt]
- project_path: [project root path]
```

The image is saved to the project's `public/` directory.

### 5. Show Usage in Remotion

```tsx
import { Img, staticFile } from "remotion";

// As a background
<Img
  src={staticFile("images/generated-background.png")}
  style={{ width: "100%", height: "100%", objectFit: "cover" }}
/>

// As an element with animation
const frame = useCurrentFrame();
const scale = spring({ frame, fps, config: { damping: 200 } });
<Img
  src={staticFile("images/product.png")}
  style={{ transform: `scale(${scale})` }}
/>
```

### 6. Iterate

If the image isn't right:
- Adjust the prompt — be more specific about what's wrong
- Try a different style descriptor
- Generate multiple variations and let the user pick
- Use negative prompts if the model supports them ("no text, no watermark")
