# Model Providers — Comprehensive Reference

This rule covers every supported AI provider for each media generation function. When a command needs to generate media, read `config.yaml` to determine which provider to use, then follow the corresponding workflow below.

## Reading the Config

At the start of any media generation task, read the project's `config.yaml`:

```bash
cat config.yaml 2>/dev/null
```

If no `config.yaml` exists, suggest running `/select-models` to create one. Fall back to the `free` preset defaults if the user wants to proceed without configuring.

**Free preset defaults:**
- TTS: edge-tts
- Music: musicgen
- Image: pixazo
- Video: google-ai-studio
- Subtitle: faster-whisper
- SFX: freesound

---

## TTS Providers

### edge-tts (FREE)

Microsoft Edge neural voices via Python package. Excellent quality with Korean language support.

**Install:**
```bash
pip install edge-tts
```

**Usage:**
```bash
# Generate voiceover
edge-tts --text "[script]" --voice "[voice_id]" --write-media public/audio/voiceover.mp3

# Get audio duration
ffprobe -v error -show_entries format=duration -of csv=p=0 public/audio/voiceover.mp3
```

**Korean voices:**
| Voice ID | Gender | Style |
|---|---|---|
| ko-KR-SunHiNeural | Female | Clear, professional |
| ko-KR-InJoonNeural | Male | Warm, natural |
| ko-KR-HyunsuNeural | Male | Friendly, conversational |
| ko-KR-BongJinNeural | Male | Authoritative |
| ko-KR-GookMinNeural | Male | Youthful |
| ko-KR-JiMinNeural | Female | Bright, energetic |
| ko-KR-SeoHyeonNeural | Female | Calm, soothing |
| ko-KR-SoonBokNeural | Female | Warm, mature |
| ko-KR-YuJinNeural | Female | Professional |

**English voices:**
| Voice ID | Gender | Style |
|---|---|---|
| en-US-AriaNeural | Female | Energetic, engaging |
| en-US-GuyNeural | Male | Professional, warm |
| en-US-JennyNeural | Female | Friendly, conversational |
| en-US-DavisNeural | Male | Casual, youthful |
| en-GB-SoniaNeural | Female | British, clear |
| en-GB-RyanNeural | Male | British, professional |

**List all available voices:**
```bash
edge-tts --list-voices | grep -E "^(ko-KR|en-US|en-GB|ja-JP)"
```

**Output:** MP3 file saved to the specified path.

**Quality:** Very good. Natural prosody, good intonation. Korean pronunciation is excellent.

**Limitations:** Requires internet (uses Edge API). No voice cloning. No fine-grained emotion control.

---

### kokoro (FREE)

Lightweight 82M parameter TTS model. Fast inference, good for English.

**Install:**
```bash
pip install kokoro soundfile
```

**Usage:**
```bash
python3 -c "
from kokoro import KPipeline
import soundfile as sf

pipeline = KPipeline(lang_code='a')  # 'a' = American English
audio, sr = pipeline('[script]', voice='af_heart')
sf.write('public/audio/voiceover.wav', audio, sr)
print(f'Duration: {len(audio)/sr:.2f}s')
"
```

**Available voices:**
| Voice | Gender | Style |
|---|---|---|
| af_heart | Female | Warm, natural |
| af_bella | Female | Clear, professional |
| af_sarah | Female | Friendly |
| am_adam | Male | Professional |
| am_michael | Male | Warm |

**Output:** WAV file.

**Quality:** Good for English. Runs entirely locally with no internet required.

**Limitations:** English only — no Korean support. Smaller vocabulary than edge-tts. 82M model may struggle with complex sentences.

---

### coqui-xtts (FREE)

Voice cloning TTS. Clone any voice from a short reference audio sample.

**Install:**
```bash
pip install TTS
```

**Usage:**
```bash
python3 -c "
from TTS.api import TTS

tts = TTS('tts_models/multilingual/multi-dataset/xtts_v2')
tts.tts_to_file(
    text='[script]',
    speaker_wav='[path/to/reference_audio.wav]',
    language='ko',  # 'ko' for Korean, 'en' for English
    file_path='public/audio/voiceover.wav'
)
"
```

**Supported languages:** English, Korean, Japanese, Chinese, German, French, Spanish, Portuguese, Italian, Polish, Turkish, Russian, Dutch, Czech, Arabic, Hindi

**Output:** WAV file matching the reference voice.

**Quality:** Good voice cloning quality. Requires 6-30 seconds of clean reference audio.

**Limitations:** Slower inference (~2x realtime on CPU). Large model download (~1.8GB). Voice cloning quality depends heavily on reference audio quality.

---

### elevenlabs via KIE (PAID)

Premium ElevenLabs voices through the KIE API aggregator.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_tts:
- text: [script]
- voice: [voice name]
- project_path: [project root path]
```

**Voices:** Eric, Rachel, Aria, Roger, Sarah, Laura, Charlie, George

**Output:** Audio file saved to `public/` directory. Returns file path and duration.

**Quality:** Excellent. Industry-leading naturalness and expressiveness.

**Cost:** Included in KIE plan pricing.

---

### elevenlabs-direct (PAID)

Direct ElevenLabs API access. Gives full control including voice cloning, voice design, and audio isolation.

**Requires:** `ELEVENLABS_API_KEY`

**Usage:**
```
Use ElevenLabs MCP tools:
- text_to_speech: Generate speech
- get_voices: List available voices
- clone_voice: Create voice from audio samples
```

**Output:** Audio files saved to `public/audio/` (configured via `ELEVENLABS_MCP_BASE_PATH`).

**Quality:** Excellent. Same quality as KIE but with more control and features.

**Cost:** Free tier: 10,000 characters/month. Paid plans from $5/month.

---

## Music Providers

### musicgen (FREE)

Meta's MusicGen model. Generate music from text descriptions locally.

**Install:**
```bash
pip install transformers scipy soundfile torch
```

**Usage:**
```bash
python3 << 'PYEOF'
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import scipy.io.wavfile
import numpy as np

model_name = "facebook/musicgen-small"  # or musicgen-medium, musicgen-large
processor = AutoProcessor.from_pretrained(model_name)
model = MusicgenForConditionalGeneration.from_pretrained(model_name)

inputs = processor(
    text=["[music prompt, e.g. upbeat electronic, no vocals, modern tech vibe]"],
    padding=True,
    return_tensors="pt"
)

# max_new_tokens controls duration: 256 = ~5s, 512 = ~10s, 1024 = ~20s, 1536 = ~30s
audio_values = model.generate(**inputs, max_new_tokens=1536)
audio_data = audio_values[0, 0].cpu().numpy()

sampling_rate = model.config.audio_encoder.sampling_rate
scipy.io.wavfile.write("public/audio/music.wav", sampling_rate, audio_data)
print(f"Duration: {len(audio_data)/sampling_rate:.1f}s")
PYEOF
```

**Model sizes:**
| Model | Parameters | Quality | Speed (CPU) | Speed (GPU) |
|---|---|---|---|---|
| musicgen-small | 300M | Good | ~2min for 30s | ~15s for 30s |
| musicgen-medium | 1.5B | Very good | ~5min for 30s | ~30s for 30s |
| musicgen-large | 3.3B | Excellent | ~10min for 30s | ~1min for 30s |

**Prompt tips:**
- Always include "no vocals" or "instrumental" for background music
- Specify genre, mood, tempo, and instruments
- Example: "calm lo-fi hip hop, mellow piano and soft drums, no vocals, study music atmosphere"

**Output:** WAV file at 32kHz.

**Quality:** Good to excellent depending on model size. Instrumental music only.

**Limitations:** No vocals/singing. CPU inference is slow. First run downloads model weights (300MB-3.3GB).

---

### ace-step (FREE)

ACE-Step local music generation server. Higher quality than MusicGen, requires GPU.

**Install:**
```bash
git clone https://github.com/ace-step/ACE-Step.git
cd ACE-Step
pip install -r requirements.txt
python server.py --port 8001
```

**Usage:**
```bash
curl -X POST http://localhost:8001/generate \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[music prompt]",
    "duration": 30,
    "seed": -1
  }' \
  -o public/audio/music.wav
```

**Output:** WAV file.

**Quality:** Very good. Supports more complex compositions than MusicGen.

**Limitations:** Requires running the ACE-Step server locally. Needs GPU with 8GB+ VRAM. Setup is more involved.

---

### suno via KIE (PAID)

Suno V3.5-V5 via the KIE API. Professional-grade music generation.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_music:
- prompt: "[music description, always include 'no vocals']"
- project_path: [project root path]
```

**Output:** Audio file saved to `public/` directory.

**Quality:** Excellent. Professional studio quality. Can generate vocals if desired.

**Cost:** Included in KIE plan pricing.

---

## Image Providers

### pixazo (FREE)

Free cloud image generation API using FLUX models.

**Install:** No installation needed.

**Usage:**
```bash
curl -s -X POST "https://api.pixazo.com/v1/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[image prompt]",
    "model": "flux-schnell",
    "width": 1920,
    "height": 1080,
    "num_inference_steps": 4
  }' | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
img_data = base64.b64decode(data['images'][0])
with open('public/images/generated.png', 'wb') as f:
    f.write(img_data)
print('Image saved to public/images/generated.png')
"
```

**Models:**
- `flux-schnell` — Fast, 4 steps, good quality
- `flux-dev` — Slower, 20 steps, better quality

**Output:** PNG image.

**Quality:** Good. FLUX-schnell is fast with decent quality. FLUX-dev is slower but higher fidelity.

**Limitations:** Rate limited on free tier. May have queue during peak hours.

---

### flux-local (FREE)

Run FLUX locally via ComfyUI. Best quality, requires GPU.

**Install:**
```bash
# Install ComfyUI
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
pip install -r requirements.txt

# Download FLUX model (choose one)
# FLUX.1-schnell (~12GB): https://huggingface.co/black-forest-labs/FLUX.1-schnell
# Place in ComfyUI/models/unet/

# Start ComfyUI
python main.py --listen 0.0.0.0 --port 8188
```

**Usage:**
```bash
# Generate via ComfyUI API
python3 << 'PYEOF'
import json, urllib.request, urllib.parse, time

COMFYUI_URL = "http://127.0.0.1:8188"

workflow = {
    "prompt": {
        # FLUX workflow nodes — adapt to your ComfyUI setup
        # Use the ComfyUI web UI to create and export an API workflow
    }
}

# Queue the prompt
data = json.dumps({"prompt": workflow}).encode()
req = urllib.request.Request(f"{COMFYUI_URL}/prompt", data=data)
response = json.loads(urllib.request.urlopen(req).read())
prompt_id = response["prompt_id"]

# Poll for completion and download
# (See ComfyUI API docs for polling pattern)
PYEOF
```

**Output:** PNG image.

**Quality:** Excellent. Same quality as Replicate FLUX but free.

**Limitations:** Requires local GPU with 8GB+ VRAM (12GB+ recommended). ComfyUI setup can be complex. Not suitable for machines without GPU.

---

### replicate (PAID)

Replicate cloud API with access to top image models.

**Requires:** `REPLICATE_API_TOKEN`

**Usage (via MCP):**
```
replicate_run:
  model: "black-forest-labs/flux-1.1-pro"
  input:
    prompt: "[image prompt]"
    aspect_ratio: "16:9"
    output_format: "png"
```

Download the output:
```bash
curl -o public/images/[name].png "[replicate_output_url]"
```

**Available models:**
| Model | Cost | Best for |
|---|---|---|
| FLUX 1.1 Pro | ~$0.04 | High quality, great prompt following |
| Imagen 4 | ~$0.03 | Photorealistic images |
| Ideogram v3 | ~$0.05 | Text-in-image, logos |
| FLUX Kontext | ~$0.04 | Style matching, multi-reference |

**Quality:** Excellent. Cloud GPUs, latest models.

**Cost:** Pay-per-use, typically $0.003-$0.05 per image.

---

### kie (PAID)

KIE API image generation.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_image:
- prompt: [image prompt]
- project_path: [project root path]
```

**Output:** Image saved to `public/` directory.

**Quality:** Good. Fast and integrated.

**Cost:** Included in KIE plan pricing.

---

## Video Providers

### google-ai-studio (FREE)

Google Veo via AI Studio. Free access to cutting-edge video generation with rate limiting.

**Requires:** `GOOGLE_AI_STUDIO_KEY` (free from https://aistudio.google.com/apikey)

**Install:**
```bash
pip install google-genai
```

**Usage:**
```bash
python3 << 'PYEOF'
import time
from google import genai

client = genai.Client(api_key="[GOOGLE_AI_STUDIO_KEY or read from env]")

# Generate video
operation = client.models.generate_videos(
    model="veo-3.0-generate-preview",
    prompt="[video prompt with camera motion and scene description]",
    config=genai.types.GenerateVideosConfig(
        person_generation="allow_all",
        aspect_ratio="16:9",
        number_of_videos=1,
    ),
)

# Poll until complete
while not operation.done:
    time.sleep(20)
    operation = client.operations.get(operation)

# Download the video
video = operation.result.generated_videos[0]
video_data = client.files.download(file=video.video)
with open("public/footage/generated-clip.mp4", "wb") as f:
    f.write(video_data)

print("Video saved to public/footage/generated-clip.mp4")
PYEOF
```

**Output:** MP4 file, typically 5-8 seconds.

**Quality:** Excellent. Veo produces highly cinematic results.

**Limitations:** Rate limited (varies by tier). Free tier may have queues. Processing takes 1-5 minutes. Video length typically limited to 5-8 seconds.

---

### cogvideo-local (FREE)

CogVideoX running locally. Text-to-video and image-to-video.

**Install:**
```bash
pip install diffusers torch transformers accelerate
```

**Usage:**
```bash
python3 << 'PYEOF'
import torch
from diffusers import CogVideoXPipeline
from diffusers.utils import export_to_video

pipe = CogVideoXPipeline.from_pretrained(
    "THUDM/CogVideoX-2b",
    torch_dtype=torch.float16
)
pipe.to("cuda")
pipe.enable_model_cpu_offload()

video_frames = pipe(
    prompt="[video prompt]",
    num_frames=49,  # ~6 seconds at 8fps
    guidance_scale=6.0,
    num_inference_steps=50,
).frames[0]

export_to_video(video_frames, "public/footage/generated-clip.mp4", fps=8)
print("Video saved to public/footage/generated-clip.mp4")
PYEOF
```

**Models:**
| Model | VRAM | Quality |
|---|---|---|
| CogVideoX-2b | ~16GB | Good |
| CogVideoX-5b | ~24GB | Very good |

**Output:** MP4 file, 6-8 seconds at 8fps.

**Quality:** Good. Open-source model, results are decent but not on par with Veo or Kling.

**Limitations:** Requires GPU with 16GB+ VRAM. Inference is slow (5-15 minutes). Lower fps than commercial models. First run downloads ~10GB model.

---

### replicate video (PAID)

Replicate cloud API for video generation.

**Requires:** `REPLICATE_API_TOKEN`

**Usage (via MCP):**
```
# Text-to-Video with Wan 2.5
replicate_create_prediction:
  model: "wan-video/wan-2.5-t2v-fast"
  input:
    prompt: "[video prompt]"
    num_frames: 81
    resolution: "480p"

# Image-to-Video with Kling 2.6
replicate_create_prediction:
  model: "kwaivgi/kling-v2.6-pro"
  input:
    image: "[image_url]"
    prompt: "[motion description]"
    duration: 5
```

Poll with `replicate_get_prediction` until status is `succeeded`, then download:
```bash
curl -o public/footage/[name].mp4 "[replicate_output_url]"
```

**Available models:**
| Model | Cost | Speed | Quality |
|---|---|---|---|
| Wan 2.5 Fast | ~$0.03 | ~40s | Good |
| Wan 2.5 | ~$0.08 | ~2min | Very good |
| Kling 2.6 Pro | ~$0.50 | ~3min | Excellent |

**Quality:** Very good to excellent depending on model.

**Cost:** Pay-per-use, $0.03-$0.50 per clip.

---

### kie video (PAID)

Veo 3.1 via KIE API. Highest quality video generation.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_video:
- prompt: [video prompt]
- project_path: [project root path]
```

**Output:** Video saved to `public/` directory.

**Quality:** Excellent. Veo 3.1 is the flagship video generation model.

**Cost:** Included in KIE plan pricing.

---

## Subtitle Providers

### faster-whisper (FREE)

CTranslate2-optimized Whisper model. Fast local transcription.

**Install:**
```bash
pip install faster-whisper
```

**Usage:**
```bash
python3 << 'PYEOF'
from faster_whisper import WhisperModel

model = WhisperModel("large-v3-turbo", device="cpu", compute_type="int8")
# For GPU: device="cuda", compute_type="float16"

segments, info = model.transcribe("[audio_file_path]", word_timestamps=True)
print(f"Language: {info.language} (probability: {info.language_probability:.2f})")

# Generate SRT file
with open("public/subtitles/captions.srt", "w", encoding="utf-8") as f:
    for i, segment in enumerate(segments, 1):
        start_h = int(segment.start // 3600)
        start_m = int((segment.start % 3600) // 60)
        start_s = int(segment.start % 60)
        start_ms = int((segment.start % 1) * 1000)
        
        end_h = int(segment.end // 3600)
        end_m = int((segment.end % 3600) // 60)
        end_s = int(segment.end % 60)
        end_ms = int((segment.end % 1) * 1000)
        
        f.write(f"{i}\n")
        f.write(f"{start_h:02d}:{start_m:02d}:{start_s:02d},{start_ms:03d} --> ")
        f.write(f"{end_h:02d}:{end_m:02d}:{end_s:02d},{end_ms:03d}\n")
        f.write(f"{segment.text.strip()}\n\n")

print("SRT saved to public/subtitles/captions.srt")
PYEOF
```

**Models:**
| Model | Size | Quality | Speed (CPU) | Speed (GPU) |
|---|---|---|---|---|
| tiny | 39M | Basic | Very fast | - |
| base | 74M | Fair | Fast | - |
| small | 244M | Good | Medium | Fast |
| medium | 769M | Very good | Slow | Medium |
| large-v3 | 1.5GB | Excellent | Very slow | Medium |
| large-v3-turbo | 809M | Excellent | Medium | Fast |

**Recommended:** `large-v3-turbo` — best quality-to-speed ratio.

**Output:** SRT file with timestamps. Also supports word-level timestamps for animated captions.

**Quality:** Excellent. State-of-the-art transcription for 100+ languages including Korean.

**Limitations:** First run downloads model weights. CPU inference for large models can be slow.

---

### whisper-local (FREE)

Remotion's built-in Whisper.cpp integration. Integrated into the Remotion toolchain.

**Install:**
```bash
npx remotion add @remotion/install-whisper-cpp
```

**Usage:**
```typescript
import { installWhisperCpp, transcribe } from "@remotion/install-whisper-cpp";

// One-time install
await installWhisperCpp({ version: "1.5.5" });

// Transcribe
const result = await transcribe({
  inputPath: "public/audio/voiceover.mp3",
  whisperPath: ".whisper",
  model: "medium",  // tiny, base, small, medium, large
  tokenLevelTimestamps: true,
});
```

**Output:** JSON with word-level timestamps. Native integration with `@remotion/captions`.

**Quality:** Good. Same underlying Whisper model but via whisper.cpp (C++ implementation).

**Limitations:** Fewer model options than faster-whisper. Setup is Remotion-specific.

---

### kie subtitle (PAID)

Cloud Whisper via KIE API.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_subtitles:
- input: [path to audio/video file]
- project_path: [project root path]
```

**Output:** SRT file saved to project.

**Quality:** Excellent. Fast cloud processing, no local compute needed.

**Cost:** Included in KIE plan pricing.

---

## SFX Providers

### freesound (FREE)

Freesound.org — massive library of user-contributed sound effects.

**Usage:**
```bash
# Search for a sound effect
curl -s "https://freesound.org/apiv2/search/text/?query=[keyword]&token=[FREESOUND_API_KEY]&fields=id,name,previews,duration,license" | python3 -c "
import sys, json
data = json.load(sys.stdin)
for sound in data['results'][:5]:
    print(f\"  {sound['name']} ({sound['duration']:.1f}s) - {sound['license']}\")
    print(f\"    Preview: {sound['previews']['preview-hq-mp3']}\")
"

# Download a specific sound
curl -o public/audio/sfx/[name].mp3 "[preview_url]"
```

**Alternative (no API key):** Search on https://freesound.org manually, download, and place in `public/audio/sfx/`.

**Output:** MP3/WAV sound effect files.

**Quality:** Varies widely. Professional-quality sounds are available but require searching.

**Limitations:** Requires attribution for most sounds (Creative Commons license). Quality varies. Search results may not always match exactly.

---

### elevenlabs SFX via KIE (PAID)

AI-generated sound effects from text descriptions via KIE.

**Requires:** `KIE_API_KEY`

**Usage:**
```
Use remotion-media generate_sfx:
- text: [sound effect description, e.g. "glass breaking on a hard floor"]
- duration_seconds: [length]
- project_path: [project root path]
```

**Output:** Audio file saved to `public/` directory.

**Quality:** Excellent. Custom AI-generated effects that match your exact description.

**Cost:** Included in KIE plan pricing.

---

### elevenlabs-direct SFX (PAID)

Direct ElevenLabs SFX generation.

**Requires:** `ELEVENLABS_API_KEY`

**Usage:**
```
Use ElevenLabs MCP sound_generation tool:
- text: [sound effect description]
- duration_seconds: [length]
```

**Output:** Audio file saved to `public/audio/`.

**Quality:** Excellent. Same engine as KIE but with direct API control.

**Cost:** Uses ElevenLabs credits.

---

## Provider Decision Quick Reference

| Function | Free Pick | Budget Pick | Premium Pick |
|---|---|---|---|
| TTS | edge-tts | edge-tts | elevenlabs (KIE) |
| Music | musicgen | musicgen | suno (KIE) |
| Image | pixazo | replicate FLUX | kie |
| Video | google-ai-studio | replicate Wan 2.5 | kie (Veo 3.1) |
| Subtitle | faster-whisper | faster-whisper | kie |
| SFX | freesound | freesound | elevenlabs (KIE) |

## Adding New Providers

To add a new provider:

1. Add an entry to this rule file under the appropriate function section
2. Document: install, usage (bash/python), output format, quality, cost, limitations
3. Add the provider option to the relevant command files
4. Add it as an option in `commands/select-models.md`
5. Update `config.yaml` comments to include the new provider name
