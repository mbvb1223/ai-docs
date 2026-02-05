# CogVideoX-3

CogVideoX-3 is Z.AI's video generation model featuring enhanced frame generation with improved stability and clarity. The model excels at handling dynamic subject movement, instruction adherence, and realistic simulations.

## Key Specifications

| Aspect | Details |
|--------|---------|
| **Pricing** | $0.2 per video |
| **Input** | Image, Text, Start/End Frames |
| **Output** | Video |
| **Max Resolution** | Up to 4K (3840x2160) |
| **Frame Rates** | 30 or 60 fps |

## Primary Use Cases

- **Advertising & Marketing**: Generate dynamic ads with scene transitions and realistic lighting from product images or copy
- **Short Video Creation**: Convert single images or text scripts into smooth animations in realistic or 3D styles
- **Tourism Promotion**: Transform scenic photos and promotional text into immersive travel videos
- **Film & TV Production**: Generate dynamic preview clips from storyboards with camera movements and physical interactions

## Key Features

### Improved Subject Clarity
Videos feature clear subjects with stable frames, reduced distortion, and extensive subject movement support for natural, fluid animation.

### Command Compliance & Physics
The model deeply understands the intent of text commands and accurately reproduces creative requirements, handling both character actions and natural phenomena with real-world logic.

### Scene Rendering
Delivers high-definition realistic photography-style textures and precise 3D-style scene rendering across multiple aesthetic approaches.

### Frame Interpolation
Supports start and end frame inputs to generate seamless transition content, connecting static frames into dynamic narratives.

## Quick Start Examples

### Installation

```bash
pip install zai-sdk

# Verify installation
import zai
print(zai.__version__)
```

### Text-to-Video Generation

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.videos.generations(
    model="cogvideox-3",
    prompt="A cat is playing with a ball.",
    quality="quality",
    with_audio=True,
    size="1920x1080",
    fps=30,
)

# Poll for result
result = client.videos.retrieve_videos_result(id=response.id)
print(result.video_url)
```

### Image-to-Video Generation

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")
image_url = "https://example.com/image.jpg"

response = client.videos.generations(
    model="cogvideox-3",
    image_url=image_url,
    prompt="Make the picture move.",
    quality="quality",
    with_audio=True,
    size="1920x1080",
    fps=30,
)

result = client.videos.retrieve_videos_result(id=response.id)
print(result.video_url)
```

### Start and End Frame Generation

Generate videos that transition between two specific frames:

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.videos.generations(
    model="cogvideox-3",
    image_url=[
        "https://example.com/frame1.jpg",
        "https://example.com/frame2.jpg"
    ],
    prompt="Animate the scene with smooth transitions",
    quality="quality",
    with_audio=True,
    size="1920x1080",
    fps=30,
)

result = client.videos.retrieve_videos_result(id=response.id)
print(result.video_url)
```

## Configuration Options

### Quality Mode

```python
# Prioritize output quality
response = client.videos.generations(
    model="cogvideox-3",
    prompt="A beautiful sunset over the ocean",
    quality="quality",  # Higher quality, slower
    size="1920x1080",
    fps=30,
)

# Prioritize generation speed
response = client.videos.generations(
    model="cogvideox-3",
    prompt="A beautiful sunset over the ocean",
    quality="speed",  # Faster, lower quality
    size="1920x1080",
    fps=30,
)
```

### Resolution Options

```python
# Full HD
response = client.videos.generations(
    model="cogvideox-3",
    prompt="City traffic at night",
    size="1920x1080",
    fps=30,
)

# 4K
response = client.videos.generations(
    model="cogvideox-3",
    prompt="Mountain landscape",
    size="3840x2160",
    fps=30,
)
```

### Frame Rate Options

```python
# Standard 30 fps
response = client.videos.generations(
    model="cogvideox-3",
    prompt="Slow motion water droplet",
    size="1920x1080",
    fps=30,
)

# High frame rate 60 fps
response = client.videos.generations(
    model="cogvideox-3",
    prompt="Fast action sports scene",
    size="1920x1080",
    fps=60,
)
```

### Audio Generation

```python
# With audio
response = client.videos.generations(
    model="cogvideox-3",
    prompt="A thunderstorm over a city",
    with_audio=True,
    size="1920x1080",
    fps=30,
)

# Without audio
response = client.videos.generations(
    model="cogvideox-3",
    prompt="Silent film style comedy",
    with_audio=False,
    size="1920x1080",
    fps=30,
)
```

## Polling for Results

Video generation is asynchronous. Poll for completion:

```python
import time
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.videos.generations(
    model="cogvideox-3",
    prompt="A rocket launching into space",
    quality="quality",
    size="1920x1080",
    fps=30,
)

# Poll until complete
while True:
    result = client.videos.retrieve_videos_result(id=response.id)
    if result.status == "completed":
        print(f"Video URL: {result.video_url}")
        break
    elif result.status == "failed":
        print(f"Generation failed: {result.error}")
        break
    time.sleep(5)  # Wait 5 seconds before checking again
```

## Installation

**Python:**
```bash
pip install zai-sdk
```

**Java (Maven):**
```xml
<dependency>
    <groupId>ai.z.openapi</groupId>
    <artifactId>zai-sdk</artifactId>
    <version>0.3.0</version>
</dependency>
```
