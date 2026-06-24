# Blog Background Generation — SDXL via REST API

A fully self-contained approach for generating blog hero/background images
directly through ComfyUI's REST API — no skill scripts, no comfy-cli, just
`urllib.request`. Works on Windows through `execute_code`.

## Workflow: SDXL txt2img → blog background

Resolution: 1344×768 (16:9). Model: Juggernaut XL v9 (or any SDXL checkpoint).

### Step 1: Check what the running server sees

```python
import urllib.request, json

req = urllib.request.Request("http://localhost:8188/object_info/CheckpointLoaderSimple")
with urllib.request.urlopen(req, timeout=5) as resp:
    data = json.loads(resp.read())
    ckpts = data["CheckpointLoaderSimple"]["input"]["required"]["ckpt_name"][0]
    for c in ckpts:
        print(f"  📦 {c}")
```

### Step 2: Build and submit workflow

```python
import urllib.request, json, random

seed = random.randint(1, 2**31)

workflow = {
    "3": {"class_type": "KSampler", "inputs": {
        "seed": seed, "steps": 30, "cfg": 7,
        "sampler_name": "dpmpp_2m", "scheduler": "karras", "denoise": 1.0,
        "model": ["4", 0], "positive": ["6", 0], "negative": ["7", 0],
        "latent_image": ["5", 0]
    }},
    "4": {"class_type": "CheckpointLoaderSimple", "inputs": {
        "ckpt_name": "Juggernaut-XL_v9_RunDiffusionPhoto_v2.safetensors"
    }},
    "5": {"class_type": "EmptyLatentImage", "inputs": {
        "width": 1344, "height": 768, "batch_size": 1
    }},
    "6": {"class_type": "CLIPTextEncode", "inputs": {
        "text": "YOUR POSITIVE PROMPT HERE",
        "clip": ["4", 1]
    }},
    "7": {"class_type": "CLIPTextEncode", "inputs": {
        "text": "text, watermark, signature, logo, letters, words, frame, border, dark, gloomy, busy, cluttered, complex, realistic, photograph, 3d render, harsh shadows, high contrast, neon, cyberpunk, grunge, dirty, ugly, low quality, blurry, distorted, deformed, bad anatomy",
        "clip": ["4", 1]
    }},
    "8": {"class_type": "VAEDecode", "inputs": {
        "samples": ["3", 0], "vae": ["4", 2]
    }},
    "9": {"class_type": "SaveImage", "inputs": {
        "filename_prefix": "blog_bg",
        "images": ["8", 0]
    }}
}

data = json.dumps({"prompt": workflow}).encode()
req = urllib.request.Request(
    "http://localhost:8188/prompt",
    data=data, headers={"Content-Type": "application/json"}
)
with urllib.request.urlopen(req, timeout=10) as resp:
    result = json.loads(resp.read())
    prompt_id = result["prompt_id"]
    print(f"Submitted: {prompt_id}")
```

### Step 3: Wait and retrieve output

```python
import time

# Poll until done (typical: 15-40s on RTX 4060 for 30 steps SDXL)
while True:
    time.sleep(3)
    req = urllib.request.Request("http://localhost:8188/queue")
    with urllib.request.urlopen(req, timeout=5) as resp:
        queue = json.loads(resp.read())
    if not queue.get("queue_running") and not queue.get("queue_pending"):
        break

# Get filename from history
req = urllib.request.Request(f"http://localhost:8188/history/{prompt_id}")
with urllib.request.urlopen(req, timeout=5) as resp:
    history = json.loads(resp.read())
    for node_id, out in history[prompt_id]["outputs"].items():
        for img in out.get("images", []):
            print(f"📷 {img['filename']}")
```

### Step 4: Output paths

- **ComfyUI Desktop**: `D:\Comfy-Desktop\ComfyUI-Shared\output\{filename}`
- **View in browser**: `http://localhost:8188/view?filename={filename}&type=output`

## Real prompt: Reverse 1999 British vintage dark academia background

Used for loze-blog-next (dark academia variant — deep green, antique gold, burgundy palette):

```
Reverse 1999 game art style, British vintage aesthetic, oil painting texture,
dark academia atmosphere. Muted sophisticated palette: deep forest green, antique gold,
burgundy, aged navy blue. Art deco ornamental borders, vintage clock faces, soft
mysterious lighting, foggy London ambiance. Baroque architectural elements, aged
parchment texture, subtle magical sparkles. Antique library or study room vibe,
leather-bound books, brass details, candelabra glow. Painterly brushstrokes, concept
art quality, moody and elegant, misty atmosphere. Composition with generous dark
negative space on left for text overlay. Vertical format friendly, 8k masterpiece.
```

Negative prompt: `bright, sunny, cheerful, cute, anime, cartoon, modern, minimalist,
white background, text, watermark, signature, logo, letters, frame, border, people,
faces, characters, nsfw, blurry, low quality, distorted, deformed, photograph, 3d render`

Steps: 35 (more steps for finer painterly detail). CFG: 7.5 (slightly higher for
stronger prompt adherence on complex art styles).

## Real prompt: Japanese warm illustration blog background

Used for loze-blog-next (暖琥珀 #b7610a + 奶油纸 #faf9f6 palette):

```
Japanese illustration style, soft warm lighting, Ghibli-inspired cozy atmosphere.
Pastel amber and cream palette with #b7610a warm brown accents. Gentle golden hour
light rays filtering through, abstract flowing organic shapes, subtle botanical hints,
minimalist composition with generous negative space for text. Soft watercolor texture,
dreamy bokeh background, healing and calm mood. Clean aesthetic, 8k quality,
trending on pixiv.
```

Negative prompt: `text, watermark, signature, logo, letters, words, frame, border,
dark, gloomy, busy, cluttered, complex, realistic, photograph, 3d render, harsh shadows,
high contrast, neon, cyberpunk, grunge, dirty, ugly, low quality, blurry, distorted,
deformed, bad anatomy, extra limbs, nsfw`

## Pitfalls specific to blog backgrounds

- **Negative space is critical** — the image area must leave room for text overlay.
  Keywords in positive prompt: "minimalist composition", "generous negative space",
  "clean aesthetic". In negative: "busy", "cluttered", "complex".
- **Resolution** — 16:9 (1344×768 or 1920×1080) for hero sections. Square 1024×1024
  works for card backgrounds.
- **Color lock** — include hex codes in the positive prompt (`#b7610a`, `#faf9f6`)
  to steer the palette toward the blog's accent colors.
- **No text/watermarks** — reinforce heavily in negative prompt since SDXL models
  sometimes generate pseudo-text artifacts.
