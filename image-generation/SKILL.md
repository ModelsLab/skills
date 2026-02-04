---
name: modelslab-image-generation
description: Generate high-quality AI images from text prompts or transform existing images using ModelsLab's API with 10,000+ models including FLUX, Realtime, and Community models. Supports text2img, img2img, inpainting, and ControlNet.
---

# ModelsLab Image Generation

Generate stunning AI images using ModelsLab's extensive library of models and powerful generation endpoints.

## When to Use This Skill

- Generate images from text descriptions
- Transform or modify existing images
- Fill in masked parts of images (inpainting)
- Use ControlNet for precise control
- Need fast generation (Realtime API)
- Need highest quality (Community/FLUX models)
- Create product images, marketing graphics, or artwork

## Available APIs

### 1. Realtime API (Fastest)
**Best for**: Speed-critical applications, prototypes, real-time demos
- Text2Img: `POST https://modelslab.com/api/v6/realtime/text2img`
- Img2Img: `POST https://modelslab.com/api/v6/realtime/img2img`
- Response time: < 5 seconds

### 2. Community Models (Highest Quality)
**Best for**: Production outputs, custom models, advanced features
- Text2Img: `POST https://modelslab.com/api/v6/images/text2img`
- Img2Img: `POST https://modelslab.com/api/v6/images/img2img`
- Inpainting: `POST https://modelslab.com/api/v6/images/inpaint`
- ControlNet: `POST https://modelslab.com/api/v6/images/controlnet`
- 10,000+ models available

### 3. FLUX API (Premium Quality)
**Best for**: State-of-the-art results
- Text2Img: `POST https://modelslab.com/api/v6/images/flux`
- Best-in-class image quality

## Quick Start: Text to Image

```python
import requests
import time

def generate_image(prompt, api_key, api_type="community"):
    """Generate an image from text.

    Args:
        prompt: Text description of the image
        api_key: Your ModelsLab API key
        api_type: "realtime" (fast) or "community" (quality)
    """
    if api_type == "realtime":
        url = "https://modelslab.com/api/v6/realtime/text2img"
        payload = {
            "key": api_key,
            "prompt": prompt,
            "negative_prompt": "blurry, low quality",
            "width": 512,
            "height": 512,
            "num_inference_steps": 20,
            "guidance_scale": 7.5
        }
    else:  # community
        url = "https://modelslab.com/api/v6/images/text2img"
        payload = {
            "key": api_key,
            "model_id": "midjourney",  # or any model
            "prompt": prompt,
            "negative_prompt": "blurry, low quality, distorted",
            "width": 768,
            "height": 768,
            "samples": 1,
            "num_inference_steps": 30,
            "guidance_scale": 7.5,
            "safety_checker": "yes"
        }

    response = requests.post(url, json=payload)
    data = response.json()

    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        # Poll for results (community API may be async)
        return poll_result(data["id"], api_key)
    else:
        raise Exception(f"Error: {data.get('message')}")

def poll_result(request_id, api_key, timeout=300):
    """Poll for async generation results."""
    start = time.time()
    while time.time() - start < timeout:
        resp = requests.post(
            f"https://modelslab.com/api/v6/images/fetch/{request_id}",
            json={"key": api_key}
        )
        data = resp.json()

        if data["status"] == "success":
            return data["output"][0]
        elif data["status"] == "failed":
            raise Exception(data.get("message", "Failed"))

        time.sleep(5)
    raise Exception("Timeout")

# Usage
image_url = generate_image(
    "A futuristic cityscape at sunset, cyberpunk style, 8k, highly detailed",
    "your_api_key",
    api_type="community"
)
print(f"Image: {image_url}")
```

## Image to Image Transformation

```python
def transform_image(init_image, prompt, api_key, strength=0.7):
    """Transform an existing image based on a prompt.

    Args:
        init_image: URL of the input image
        prompt: How to transform the image
        strength: 0.0-1.0, higher = more change
    """
    response = requests.post(
        "https://modelslab.com/api/v6/images/img2img",
        json={
            "key": api_key,
            "model_id": "midjourney",
            "prompt": prompt,
            "init_image": init_image,
            "strength": strength,
            "width": 512,
            "height": 512,
            "num_inference_steps": 30,
            "guidance_scale": 7.5
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_result(data["id"], api_key)

# Transform a photo into a painting
result = transform_image(
    "https://example.com/photo.jpg",
    "An oil painting in Van Gogh style",
    "your_api_key",
    strength=0.8
)
```

## Inpainting (Fill Masked Regions)

```python
def inpaint_image(image_url, mask_url, prompt, api_key):
    """Fill in masked parts of an image.

    Args:
        image_url: Original image URL
        mask_url: Mask image URL (white = inpaint, black = keep)
        prompt: What to generate in masked area
    """
    response = requests.post(
        "https://modelslab.com/api/v6/images/inpaint",
        json={
            "key": api_key,
            "model_id": "midjourney",
            "init_image": image_url,
            "mask_image": mask_url,
            "prompt": prompt,
            "negative_prompt": "blurry, low quality",
            "width": 512,
            "height": 512,
            "num_inference_steps": 30
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_result(data["id"], api_key)

# Replace object in image
result = inpaint_image(
    "https://example.com/room.jpg",
    "https://example.com/mask.jpg",
    "A modern red sofa",
    "your_api_key"
)
```

## ControlNet (Precise Control)

```python
def generate_with_controlnet(image_url, prompt, controlnet_type, api_key):
    """Use ControlNet for precise composition control.

    ControlNet Types:
        - canny: Edge detection
        - depth: Depth map
        - pose: Human pose
        - scribble: Sketch-based
        - normal: Surface normals
        - seg: Semantic segmentation
    """
    response = requests.post(
        "https://modelslab.com/api/v6/images/controlnet",
        json={
            "key": api_key,
            "model_id": "midjourney",
            "controlnet_model": controlnet_type,
            "controlnet_type": controlnet_type,
            "init_image": image_url,
            "prompt": prompt,
            "width": 512,
            "height": 512,
            "num_inference_steps": 30
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_result(data["id"], api_key)

# Preserve pose, change style
result = generate_with_controlnet(
    "https://example.com/person.jpg",
    "A superhero in dynamic pose, comic book style",
    "pose",
    "your_api_key"
)
```

## Popular Model IDs

### Photorealistic
- `realistic-vision-v13` - High-quality realistic images
- `deliberate-v2` - Versatile realistic style
- `absolute-reality-v1.6` - Ultra-realistic outputs

### Artistic/Stylized
- `midjourney` - MidJourney-style images (most popular)
- `dreamshaper-8` - Artistic and versatile
- `openjourney` - Open-source MidJourney alternative

### Anime/Illustration
- `anything-v5` - Anime style
- `counterfeit-v2.5` - Anime portraits
- `meinamix` - High-quality anime art

Browse all models: https://modelslab.com/models

## Key Parameters

| Parameter | Description | Recommended Values |
|-----------|-------------|-------------------|
| `prompt` | Text description of desired image | Be specific and detailed |
| `negative_prompt` | What to avoid | "blurry, low quality, distorted" |
| `width` / `height` | Dimensions (divisible by 8) | 512, 768, 1024 |
| `samples` | Number of images | 1-4 |
| `num_inference_steps` | Quality (higher = better) | 20 (fast), 30 (balanced), 50 (quality) |
| `guidance_scale` | Prompt adherence | 7-8 (balanced), 10-15 (strict) |
| `seed` | Reproducibility | `null` (random) or number |
| `strength` | img2img change amount | 0.3 (subtle), 0.7 (moderate), 0.9 (heavy) |
| `safety_checker` | NSFW filter | "yes" or "no" |

## Best Practices

### 1. Craft Effective Prompts
```
✗ Bad: "a car"
✓ Good: "A red Ferrari sports car, studio lighting, highly detailed, 4k, photorealistic"

Include: Subject, style, lighting, quality descriptors
```

### 2. Use Negative Prompts
```python
negative_prompt = "blurry, low quality, distorted, deformed, ugly, bad anatomy, extra limbs"
```

### 3. Choose Right API
- **Realtime**: Demos, prototypes, speed-critical
- **Community**: Production, custom styles, quality
- **FLUX**: When you need absolute best quality

### 4. Handle Async Operations
```python
# Always check status and poll if needed
if data["status"] == "processing":
    result = poll_result(data["id"], api_key)
```

### 5. Use Webhooks for Long Operations
```python
payload = {
    "key": api_key,
    "prompt": "...",
    "webhook": "https://yourserver.com/webhook",
    "track_id": "unique-identifier"
}
```

## Common Use Cases

### Product Photography
```python
image = generate_image(
    "Professional product photo of wireless headphones, white background, studio lighting, commercial photography",
    api_key,
    negative_prompt="shadows, cluttered background, low quality"
)
```

### Marketing Graphics
```python
image = generate_image(
    "Modern tech startup office, collaborative workspace, bright and airy, professional photography, 8k",
    api_key
)
```

### Artistic Creations
```python
image = generate_image(
    "A serene Japanese garden at sunset, cherry blossoms, koi pond, peaceful atmosphere, oil painting style",
    api_key,
    api_type="community"
)
```

### Image Variations (Same Seed)
```python
seed = 12345
for i in range(4):
    image = generate_image(
        f"A cozy coffee shop interior, variation {i+1}",
        api_key,
        seed=seed
    )
```

## Error Handling

```python
try:
    image = generate_image(prompt, api_key)
    print(f"Success: {image}")
except Exception as e:
    print(f"Generation failed: {e}")
    # Log error, retry, or notify user
```

## Resources

- **API Documentation**: https://docs.modelslab.com/image-generation/overview
- **Model Library**: https://modelslab.com/models
- **Get API Key**: https://modelslab.com/dashboard
- **Community**: https://discord.gg/modelslab
- **Support**: support@modelslab.com

## Related Skills

- `modelslab-image-editing` - Edit and enhance images
- `modelslab-video-generation` - Generate videos from images
- `modelslab-webhooks` - Async operation handling
- `modelslab-sdk-usage` - Use official SDKs
