---
name: modelslab-video-generation
description: Generate videos from text prompts or animate static images using ModelsLab's Video API. Supports text2video, img2video with models like CogVideoX.
---

# ModelsLab Video Generation

Generate AI videos from text descriptions or animate static images using state-of-the-art video generation models.

## When to Use This Skill

- Generate videos from text descriptions
- Animate static images
- Create short-form content
- Build video marketing materials
- Generate animations for prototypes
- Create visual storytelling content

## Available APIs

### Standard API
- **Text to Video**: `POST https://modelslab.com/api/v6/video/text2video`
- **Image to Video**: `POST https://modelslab.com/api/v6/video/img2video`

### Ultra API (HD Quality)
- **Text to Video Ultra**: `POST https://modelslab.com/api/v6/video/text2video_ultra`
- **Image to Video Ultra**: `POST https://modelslab.com/api/v6/video/img2video_ultra`

### Enterprise API
- `POST https://modelslab.com/api/v1/enterprise/video/text2video`
- `POST https://modelslab.com/api/v1/enterprise/video/img2video`

## Text to Video Pattern

```python
import requests
import time

def generate_video(prompt, api_key, num_frames=25):
    """Generate a video from a text prompt.

    Args:
        prompt: Text description of the video
        api_key: Your ModelsLab API key
        num_frames: Number of frames (16-25, more = longer video)
    """
    # Submit the generation request
    response = requests.post(
        "https://modelslab.com/api/v6/video/text2video",
        json={
            "key": api_key,
            "model_id": "cogvideox",  # CogVideoX model
            "prompt": prompt,
            "negative_prompt": "low quality, blurry, static, distorted",
            "width": 512,
            "height": 512,
            "num_frames": num_frames,
            "num_inference_steps": 20,
            "guidance_scale": 7,
            "fps": 8,
            "output_type": "mp4"
        }
    )

    data = response.json()

    if data["status"] == "error":
        raise Exception(f"Error: {data['message']}")

    if data["status"] == "success":
        return data["output"][0]

    # Video generation is async - poll for results
    request_id = data["id"]
    print(f"Video processing... Request ID: {request_id}")
    print(f"Estimated time: {data.get('eta', 'unknown')} seconds")

    return poll_video_result(request_id, api_key)

def poll_video_result(request_id, api_key, timeout=600):
    """Poll for video generation results."""
    start_time = time.time()

    while time.time() - start_time < timeout:
        fetch = requests.post(
            f"https://modelslab.com/api/v6/video/fetch/{request_id}",
            json={"key": api_key}
        )
        result = fetch.json()

        if result["status"] == "success":
            return result["output"][0]
        elif result["status"] == "failed":
            raise Exception(result.get("message", "Generation failed"))

        # Still processing
        print(f"Status: processing... ({int(time.time() - start_time)}s elapsed)")
        time.sleep(10)  # Check every 10 seconds

    raise Exception("Timeout waiting for video generation")

# Usage
video_url = generate_video(
    "A spaceship flying through an asteroid field, cinematic, 4K",
    "your_api_key",
    num_frames=25
)
print(f"Video ready: {video_url}")
```

## Image to Video (Animate Images)

```python
def animate_image(image_url, prompt, api_key, num_frames=25):
    """Animate a static image based on a motion prompt.

    Args:
        image_url: URL of the image to animate
        prompt: Description of desired motion/animation
        num_frames: Number of frames
    """
    response = requests.post(
        "https://modelslab.com/api/v6/video/img2video",
        json={
            "key": api_key,
            "model_id": "cogvideox",
            "init_image": image_url,
            "prompt": prompt,
            "negative_prompt": "static, still, low quality, blurry",
            "width": 512,
            "height": 512,
            "num_frames": num_frames,
            "num_inference_steps": 20,
            "guidance_scale": 7,
            "fps": 8,
            "output_type": "mp4"
        }
    )

    data = response.json()

    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_video_result(data["id"], api_key)
    else:
        raise Exception(data.get("message", "Unknown error"))

# Animate a landscape
video = animate_image(
    "https://example.com/landscape.jpg",
    "The clouds moving slowly across the sky, birds flying in the distance, gentle breeze",
    "your_api_key",
    num_frames=25
)
print(f"Animated video: {video}")
```

## HD Video Generation (Ultra API)

```python
def generate_hd_video(prompt, api_key):
    """Generate high-definition video."""
    response = requests.post(
        "https://modelslab.com/api/v6/video/text2video_ultra",
        json={
            "key": api_key,
            "prompt": prompt,
            "negative_prompt": "low quality, blurry, distorted",
            "num_frames": 25,
            "num_inference_steps": 30,  # More steps for HD
            "guidance_scale": 7.5
        }
    )

    data = response.json()
    if data["status"] == "processing":
        return poll_video_result(data["id"], api_key, timeout=900)  # Longer timeout for HD
    elif data["status"] == "success":
        return data["output"][0]
```

## Using Webhooks for Async Generation

```python
def generate_video_with_webhook(prompt, api_key, webhook_url, track_id):
    """Generate video and get results via webhook."""
    response = requests.post(
        "https://modelslab.com/api/v6/video/text2video",
        json={
            "key": api_key,
            "model_id": "cogvideox",
            "prompt": prompt,
            "negative_prompt": "low quality, blurry",
            "num_frames": 25,
            "webhook": webhook_url,  # Your endpoint
            "track_id": track_id     # Unique identifier
        }
    )

    data = response.json()
    print(f"Request submitted: {data['id']}")
    print(f"Will notify: {webhook_url}")
    return data["id"]

# Usage
request_id = generate_video_with_webhook(
    "A sunset over the ocean with waves crashing",
    "your_api_key",
    "https://yourserver.com/webhook/video",
    "video_001"
)
```

## Key Parameters

| Parameter | Description | Recommended Values |
|-----------|-------------|-------------------|
| `prompt` | Text description of video content | Be specific about motion and scene |
| `negative_prompt` | What to avoid | "static, low quality, blurry" |
| `model_id` | Video generation model | `cogvideox` (default) |
| `num_frames` | Video length | 16 (short), 25 (standard) |
| `width` / `height` | Video dimensions | 512x512 (standard) |
| `num_inference_steps` | Quality vs speed | 20 (fast), 30 (quality) |
| `guidance_scale` | Prompt adherence | 6-8 |
| `fps` | Frames per second | 8 (standard), 16 (smooth) |
| `output_type` | Video format | `mp4` or `gif` |

## Best Practices

### 1. Write Motion-Focused Prompts
```
✗ Bad: "A cat"
✓ Good: "A cat walking through a garden, looking around curiously, sunlight filtering through trees"

Include: Action, movement, camera motion, atmosphere
```

### 2. Set Realistic Expectations
- Videos are 2-4 seconds typically (16-25 frames)
- Generation takes 3-10 minutes depending on settings
- Best for short clips, not full productions

### 3. Use Appropriate API
- **Standard**: Good quality, reasonable speed
- **Ultra**: HD quality, takes longer
- **Enterprise**: Best performance with dedicated resources

### 4. Handle Async Operations
```python
# Video generation is ALWAYS async
# Always implement polling or use webhooks
if data["status"] == "processing":
    video = poll_video_result(data["id"], api_key)
```

### 5. Optimize Parameters for Speed
```python
# Faster generation
{
    "num_frames": 16,         # Fewer frames
    "num_inference_steps": 15, # Fewer steps
    "width": 480,             # Smaller size
    "height": 480
}

# Better quality (slower)
{
    "num_frames": 25,
    "num_inference_steps": 30,
    "width": 512,
    "height": 512
}
```

## Common Use Cases

### Product Demonstration
```python
video = generate_video(
    "Product slowly rotating on a pedestal, studio lighting, professional commercial",
    api_key,
    num_frames=20
)
```

### Social Media Content
```python
video = generate_video(
    "Trendy fashion model walking on runway, camera following, dynamic lighting, stylish",
    api_key
)
```

### Animated Landscapes
```python
video = animate_image(
    "https://example.com/mountain-scene.jpg",
    "Time lapse of clouds moving over mountains, changing lighting from dawn to dusk",
    api_key,
    num_frames=25
)
```

### Abstract Animations
```python
video = generate_video(
    "Abstract colorful particles flowing and morphing, smooth motion, vibrant colors, 4K",
    api_key
)
```

## Error Handling

```python
try:
    video = generate_video(prompt, api_key)
    print(f"Video generated: {video}")
except Exception as e:
    print(f"Video generation failed: {e}")
    # Log error, retry with different parameters, notify user
```

## Fetching Existing Requests

```python
def fetch_video_status(request_id, api_key):
    """Check status of an existing video generation request."""
    response = requests.post(
        f"https://modelslab.com/api/v6/video/fetch/{request_id}",
        json={"key": api_key}
    )

    data = response.json()
    return {
        "status": data["status"],
        "output": data.get("output", []),
        "message": data.get("message", ""),
        "eta": data.get("eta", None)
    }
```

## Performance Tips

1. **Use Webhooks**: Don't poll continuously, use webhooks
2. **Cache Results**: Store generated videos, reuse when possible
3. **Queue Requests**: Generate videos in background jobs
4. **Optimize Prompts**: Test prompts to find what works best
5. **Monitor Costs**: Video generation uses more credits than images

## Supported Models

- `cogvideox` - CogVideoX model (recommended)
- More models may be available - check documentation

## Resources

- **API Documentation**: https://docs.modelslab.com/video-api/overview
- **Text2Video Docs**: https://docs.modelslab.com/video-api/text-to-video
- **Image2Video Docs**: https://docs.modelslab.com/video-api/image-to-video
- **Get API Key**: https://modelslab.com/dashboard
- **Webhooks Guide**: See `modelslab-webhooks` skill

## Related Skills

- `modelslab-image-generation` - Generate images for img2video
- `modelslab-webhooks` - Handle async operations efficiently
- `modelslab-sdk-usage` - Use official SDKs for easier integration
