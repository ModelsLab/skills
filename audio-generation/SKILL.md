---
name: modelslab-audio-generation
description: Generate speech, music, and sound effects using ModelsLab's Audio API. Supports text-to-speech with voice cloning, voice-to-voice conversion, music generation, and speech-to-text transcription.
---

# ModelsLab Audio Generation

Generate high-quality audio including speech, music, voice cloning, and sound effects using AI.

## When to Use This Skill

- Convert text to natural-sounding speech (TTS)
- Clone voices from audio samples
- Transform voice characteristics (voice-to-voice)
- Generate music from text prompts
- Create sound effects
- Transcribe speech to text
- Build voice assistants or audiobooks
- Create podcast content

## Available APIs

### Text to Speech
- `POST https://modelslab.com/api/v6/audio/text_to_speech`
- Convert text to natural speech with various voices

### Voice Cloning (Text to Audio)
- `POST https://modelslab.com/api/v6/audio/text_to_audio`
- Clone voices and generate speech

### Voice to Voice
- `POST https://modelslab.com/api/v6/audio/voice_to_voice`
- Transform voice characteristics

### Music Generation
- `POST https://modelslab.com/api/v6/audio/music_gen`
- Generate music from text descriptions

### Sound Effects (SFX)
- `POST https://modelslab.com/api/v6/audio/sfx_gen`
- Create sound effects from text

### Speech to Text
- `POST https://modelslab.com/api/v6/audio/speech_to_text`
- Transcribe audio to text

## Text to Speech (Basic)

```python
import requests

def text_to_speech(text, api_key, voice="alloy", language="en"):
    """Convert text to speech.

    Args:
        text: The text to convert to speech
        api_key: Your ModelsLab API key
        voice: Voice ID (alloy, echo, fable, onyx, nova, shimmer, madison, etc.)
        language: Language code (en, es, fr, de, etc.)
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/text_to_speech",
        json={
            "key": api_key,
            "text": text,
            "voice_id": voice,
            "language": language
        }
    )

    data = response.json()

    if data["status"] == "success":
        return data["output"][0]  # Audio file URL
    else:
        raise Exception(f"Error: {data.get('message', 'Unknown error')}")

# Usage
audio_url = text_to_speech(
    "Hello! Welcome to ModelsLab. This is a sample of our text-to-speech API.",
    "your_api_key",
    voice="madison",
    language="english"
)
print(f"Audio URL: {audio_url}")
```

## Voice Cloning

```python
def clone_voice_and_speak(text, voice_audio_url, api_key, emotion="neutral"):
    """Clone a voice and generate speech.

    Args:
        text: Text to speak
        voice_audio_url: URL of audio to clone (4-30 seconds)
        api_key: Your API key
        emotion: neutral, happy, sad, angry, dull
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/text_to_audio",
        json={
            "key": api_key,
            "prompt": text,
            "init_audio": voice_audio_url,
            "language": "english",
            "emotion": emotion,
            "base64": False
        }
    )

    data = response.json()

    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        # Poll for results
        return poll_audio_result(data["id"], api_key)
    else:
        raise Exception(data.get("message"))

# Clone voice and speak
cloned_audio = clone_voice_and_speak(
    "This is a test of voice cloning technology.",
    "https://example.com/voice-sample.mp3",
    "your_api_key",
    emotion="happy"
)
```

## Voice to Voice Transformation

```python
def transform_voice(input_audio_url, target_voice_url, api_key):
    """Transform voice characteristics from one audio to another.

    Args:
        input_audio_url: Original audio
        target_voice_url: Voice to transform into
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/voice_to_voice",
        json={
            "key": api_key,
            "init_audio": input_audio_url,
            "target_audio": target_voice_url
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_audio_result(data["id"], api_key)
```

## Music Generation

```python
def generate_music(prompt, api_key, duration=10):
    """Generate music from a text description.

    Args:
        prompt: Description of music style/mood
        duration: Length in seconds (5-30)
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/music_gen",
        json={
            "key": api_key,
            "prompt": prompt,
            "duration": duration
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_audio_result(data["id"], api_key)

# Generate background music
music_url = generate_music(
    "Upbeat electronic music with a driving beat, perfect for a tech startup video",
    "your_api_key",
    duration=15
)
print(f"Music: {music_url}")
```

## Sound Effects Generation

```python
def generate_sound_effect(description, api_key, duration=5):
    """Generate a sound effect from description.

    Args:
        description: What sound to generate
        duration: Length in seconds
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/sfx_gen",
        json={
            "key": api_key,
            "prompt": description,
            "duration": duration
        }
    )

    data = response.json()
    if data["status"] == "success":
        return data["output"][0]
    elif data["status"] == "processing":
        return poll_audio_result(data["id"], api_key)

# Generate door slam sound
sfx_url = generate_sound_effect(
    "Heavy wooden door slamming shut",
    "your_api_key",
    duration=2
)
```

## Speech to Text (Transcription)

```python
def transcribe_audio(audio_url, api_key, language="en"):
    """Transcribe speech from audio to text.

    Args:
        audio_url: URL of audio file
        language: Language code (en, es, fr, etc.)
    """
    response = requests.post(
        "https://modelslab.com/api/v6/audio/speech_to_text",
        json={
            "key": api_key,
            "audio": audio_url,
            "language": language
        }
    )

    data = response.json()

    if data["status"] == "success":
        return data["text"]  # Transcribed text
    else:
        raise Exception(data.get("message"))

# Transcribe audio
text = transcribe_audio(
    "https://example.com/speech.mp3",
    "your_api_key",
    language="en"
)
print(f"Transcription: {text}")
```

## Polling for Async Results

```python
import time

def poll_audio_result(request_id, api_key, timeout=300):
    """Poll for async audio generation results."""
    start_time = time.time()

    while time.time() - start_time < timeout:
        fetch = requests.post(
            f"https://modelslab.com/api/v6/audio/fetch/{request_id}",
            json={"key": api_key}
        )
        result = fetch.json()

        if result["status"] == "success":
            return result["output"][0]
        elif result["status"] == "failed":
            raise Exception(result.get("message", "Generation failed"))

        time.sleep(5)

    raise Exception("Timeout waiting for audio generation")
```

## Available Voice IDs

### English Voices
- `alloy` - Neutral, versatile
- `echo` - Male, clear
- `fable` - Expressive, storytelling
- `onyx` - Deep, authoritative
- `nova` - Energetic, friendly
- `shimmer` - Soft, gentle
- `madison` - Professional female

### Custom Voices
You can also upload and use custom voice samples for cloning.

## Key Parameters

| Parameter | Description | Values |
|-----------|-------------|--------|
| `text` / `prompt` | Text content or description | Any text |
| `voice_id` | Pre-trained voice | alloy, madison, etc. |
| `init_audio` | Audio to clone (4-30 sec) | Audio URL |
| `language` | Language of speech | en, es, fr, de, etc. |
| `emotion` | Emotional tone | neutral, happy, sad, angry, dull |
| `duration` | Audio length (music/SFX) | 5-30 seconds |
| `base64` | Return base64 encoded | true / false |
| `webhook` | Webhook URL for async | URL string |

## Best Practices

### 1. Choose Right Voice
```python
# Professional narration
voice = "onyx"  # Deep, authoritative

# Friendly assistant
voice = "nova"  # Energetic, approachable

# Storytelling
voice = "fable"  # Expressive
```

### 2. Voice Cloning Requirements
- Audio must be 4-30 seconds long
- Clear, high-quality recording
- Single speaker
- Minimal background noise
- Consistent volume

### 3. Optimize Music Prompts
```
✗ Bad: "music"
✓ Good: "Upbeat jazz piano with walking bass, 1950s style, energetic"

Include: Genre, instruments, mood, tempo, era
```

### 4. Use Webhooks for Long Operations
```python
payload = {
    "key": api_key,
    "prompt": "...",
    "webhook": "https://yourserver.com/webhook/audio",
    "track_id": "audio_001"
}
```

### 5. Handle Different Languages
```python
# Spanish TTS
audio = text_to_speech(
    "Hola, ¿cómo estás?",
    api_key,
    voice="alloy",
    language="es"
)
```

## Common Use Cases

### Audiobook Narration
```python
chapters = ["Chapter 1 text...", "Chapter 2 text..."]
audiobook = []

for i, chapter_text in enumerate(chapters):
    audio = text_to_speech(
        chapter_text,
        api_key,
        voice="fable",
        language="en"
    )
    audiobook.append(audio)
    print(f"Chapter {i+1} generated: {audio}")
```

### Podcast Intro Music
```python
intro = generate_music(
    "Energetic podcast intro music, catchy melody, modern production",
    api_key,
    duration=10
)
```

### Voice Assistant Responses
```python
def assistant_speak(message, api_key):
    """Generate natural assistant voice."""
    return text_to_speech(
        message,
        api_key,
        voice="nova",
        language="en"
    )

response = assistant_speak(
    "I've completed your request. Is there anything else I can help with?",
    api_key
)
```

### Meeting Transcription
```python
def transcribe_meeting(meeting_audio_url, api_key):
    """Transcribe recorded meeting."""
    transcript = transcribe_audio(
        meeting_audio_url,
        api_key,
        language="en"
    )
    return transcript
```

### Game Sound Effects
```python
sounds = {
    "footstep": "Footsteps on wooden floor",
    "explosion": "Large explosion with debris",
    "sword": "Sword unsheathing, metallic sound"
}

for name, description in sounds.items():
    sfx = generate_sound_effect(description, api_key, duration=2)
    print(f"{name}: {sfx}")
```

## Error Handling

```python
try:
    audio = text_to_speech(text, api_key)
    print(f"Audio generated: {audio}")
except Exception as e:
    print(f"Audio generation failed: {e}")
    # Log error, retry, or use fallback
```

## Enterprise API

For dedicated resources and better performance:

```python
# Enterprise endpoints
url = "https://modelslab.com/api/v1/enterprise/audio/text_to_speech"
url = "https://modelslab.com/api/v1/enterprise/audio/voice_cloning"
url = "https://modelslab.com/api/v1/enterprise/audio/speech_to_text"
```

## Resources

- **Audio API Docs**: https://docs.modelslab.com/audio-api/overview
- **Voice Cloning**: https://docs.modelslab.com/voice-cloning/voice-cloning
- **TTS Docs**: https://docs.modelslab.com/voice-cloning/text-to-speech
- **Get API Key**: https://modelslab.com/dashboard
- **Voice Library**: Available in dashboard

## Related Skills

- `modelslab-video-generation` - Add audio to videos
- `modelslab-webhooks` - Handle async audio generation
- `modelslab-sdk-usage` - Use official SDKs
