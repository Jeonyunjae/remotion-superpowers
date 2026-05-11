---
name: generate-clip
description: Generate AI video clips for your Remotion project. Create unique footage from text descriptions or animate still images into video — no stock footage needed.
---

# Generate Clip — AI Video Generation

You are helping the user generate custom AI video clips for their Remotion project. This creates unique footage that doesn't exist anywhere — going beyond stock footage.

**Load the `remotion-production` skill** for the `video-generation` and `model-providers` rules.

## Workflow

### 1. Understand the Need

Ask the user:
- **What should the clip show?** — A scene, product, animation, nature, abstract?
- **Duration** — How long? (typically 3-10 seconds for AI clips)
- **Style** — Cinematic, drone shot, close-up, slow motion, time-lapse?
- **Source** — Text-to-video (from description) or image-to-video (animate a still)?

### 2. Check Provider Configuration

Read `config.yaml` to determine the video provider:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, ask the user: "Would you like to run `/select-models` to choose your video provider, or use the default (google-ai-studio, free)?"

### 3. Craft the Video Prompt

Good video generation prompts include:
- Scene description with action/motion
- Camera movement (pan, zoom, dolly, crane, static)
- Lighting and mood
- Speed (slow motion, normal, time-lapse)

**Good prompt examples:**
- "Slow overhead crane shot of a modern city at golden hour, cars moving on streets, warm cinematic lighting"
- "Close-up of coffee being poured into a white mug, steam rising, soft morning light, shallow depth of field"
- "Abstract fluid art animation, flowing metallic gold and deep blue paint, mesmerizing slow motion"
- "Drone flying through a forest canopy, sunlight filtering through leaves, cinematic nature footage"

### 4. Generate Video

Use the configured provider.

#### google-ai-studio provider (FREE):

```bash
python3 << 'PYEOF'
import time, os
from google import genai

client = genai.Client(api_key=os.environ.get("GOOGLE_AI_STUDIO_KEY"))

operation = client.models.generate_videos(
    model="veo-3.0-generate-preview",
    prompt="[crafted video prompt]",
    config=genai.types.GenerateVideosConfig(
        person_generation="allow_all",
        aspect_ratio="16:9",
        number_of_videos=1,
    ),
)

print("Generating video... (this may take 1-5 minutes)")
while not operation.done:
    time.sleep(20)
    operation = client.operations.get(operation)

video = operation.result.generated_videos[0]
video_data = client.files.download(file=video.video)

os.makedirs("public/footage", exist_ok=True)
with open("public/footage/[descriptive-name].mp4", "wb") as f:
    f.write(video_data)

print("Video saved to public/footage/[descriptive-name].mp4")
PYEOF
```

**Requirements:** `pip install google-genai` and `GOOGLE_AI_STUDIO_KEY` environment variable set.

**Note:** Google AI Studio is free but rate-limited. Processing takes 1-5 minutes. Video length is typically 5-8 seconds.

#### cogvideo-local provider (FREE, requires GPU):

```bash
python3 << 'PYEOF'
import torch, os
from diffusers import CogVideoXPipeline
from diffusers.utils import export_to_video

pipe = CogVideoXPipeline.from_pretrained(
    "THUDM/CogVideoX-2b",
    torch_dtype=torch.float16
)
pipe.to("cuda")
pipe.enable_model_cpu_offload()

print("Generating video... (this may take 5-15 minutes)")
video_frames = pipe(
    prompt="[crafted video prompt]",
    num_frames=49,  # ~6 seconds at 8fps
    guidance_scale=6.0,
    num_inference_steps=50,
).frames[0]

os.makedirs("public/footage", exist_ok=True)
export_to_video(video_frames, "public/footage/[descriptive-name].mp4", fps=8)
print("Video saved to public/footage/[descriptive-name].mp4")
PYEOF
```

**Requirements:** `pip install diffusers torch transformers accelerate`. GPU with 16GB+ VRAM.

**Note:** First run downloads ~10GB model. Inference is slow (5-15 minutes). Output is 8fps, shorter duration than commercial models.

#### replicate provider (PAID):

**Text-to-Video:**
```
# Wan 2.5 (open source, fast)
replicate_create_prediction:
  model: "wan-video/wan-2.5-t2v-fast"
  input:
    prompt: "[crafted prompt]"
    num_frames: 81
    resolution: "480p"
```

**Image-to-Video (animate a still):**
```
# Kling 2.6 (cinematic, top-tier I2V)
replicate_create_prediction:
  model: "kwaivgi/kling-v2.6-pro"
  input:
    image: "[image_url]"
    prompt: "[motion description]"
    duration: 5
```

Poll with `replicate_get_prediction` until status is `succeeded`, then download the output:
```bash
curl -o public/footage/[descriptive-name].mp4 "[replicate_output_url]"
```

**Model recommendations:**
- **Fast & cheap**: Wan 2.5 Fast (`wan-video/wan-2.5-t2v-fast`) — ~40s for 5s clip
- **High quality**: Kling 2.6 or Veo 3.1 (via KIE)
- **Image-to-video**: Kling 2.6 Pro (`kwaivgi/kling-v2.6-pro`) or Wan 2.5 I2V
- **Open source**: Wan 2.5 (`wan-video/wan-2.5-t2v`) — higher quality, slower

See `skills/remotion-production/rules/replicate-models.md` and `rules/video-generation.md` for detailed prompt engineering and model selection guidance.

#### kie provider (PAID):

```
Use remotion-media generate_video:
- prompt: [crafted prompt]
- project_path: [project root path]
```

The clip is saved to the project's `public/` directory.

### 5. Show Usage in Remotion

```tsx
import { OffthreadVideo, staticFile } from "remotion";

// Full-scene background clip
<OffthreadVideo
  src={staticFile("footage/ai-clip.mp4")}
  style={{ width: "100%", height: "100%", objectFit: "cover" }}
/>

// Trimmed clip in a sequence
<Sequence from={5 * fps} durationInFrames={3 * fps}>
  <OffthreadVideo
    src={staticFile("footage/ai-clip.mp4")}
    startFrom={0}
    style={{ width: "100%", height: "100%", objectFit: "cover" }}
  />
</Sequence>
```

### 6. Combine with Other Assets

AI-generated clips work great when layered:
- Use as backgrounds behind text overlays
- Intercut with stock footage from Pexels
- Add voiceover and music on top
- Apply subtle zoom/pan with `interpolate()` for extra dynamism

### Tips

- **Start short** — 3-5 second clips are the sweet spot for AI video
- **Be specific about camera motion** — "slow dolly zoom", "overhead crane shot"
- **Combine clips** — Generate several short clips and sequence them in Remotion
- **Image-to-video** — Generate a still with `/generate-image` first, then animate it for more control
- **Iterate** — If the first generation isn't perfect, refine the prompt
