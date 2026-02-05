# GLM-4.6V

GLM-4.6V is Z.AI's multimodal large language model with state-of-the-art performance in visual understanding among models of similar parameter scales.

## Model Variants

| Model | Positioning | Context |
|-------|-------------|---------|
| GLM-4.6V | Flagship, highest performance | 128K |
| GLM-4.6V-FlashX | Lightweight, high-speed, affordable | 128K |
| GLM-4.6V-Flash | Lightweight, completely free | 128K |

All variants accept video, image, text, and file inputs with text output.

## Key Capabilities

### Native Multimodal Tool Use

The system supports direct image and video passing as tool parameters without text conversion. Multimodal Output allows the model to comprehend tool results—charts, screenshots, product images—within reasoning chains.

### Primary Use Cases

**Image-Text Content Creation**
Processes mixed text/image documents and generates structured, illustrated content suitable for social media and knowledge bases.

**Visual Web Search**
Performs end-to-end workflows combining visual perception, online retrieval, and report generation with integrated text and visual elements.

**Frontend Development**
Converts screenshots to HTML/CSS/JS code with pixel-level accuracy and supports visual debugging through natural language instructions.

**Long-Context Understanding**
Processes ~150 document pages, 200 slide pages, or one-hour videos in single inference passes for financial analysis and video summarization.

## Performance Evaluation

Tested across 20+ benchmarks including MMBench, MathVista, and OCRBench, demonstrating state-of-the-art performance among open-source models of comparable scale in:
- Multimodal interaction
- Logical reasoning
- Long-context understanding

## Quick Start Examples

### Installation

```bash
pip install zai-sdk
```

### Basic Image Understanding

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/image.png"}
                },
                {
                    "type": "text",
                    "text": "Describe this image"
                }
            ]
        }
    ]
)
print(response.choices[0].message)
```

### Streaming Response

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/image.png"}
                },
                {
                    "type": "text",
                    "text": "What objects are in this image?"
                }
            ]
        }
    ],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end='', flush=True)
```

### Multiple Images

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/image1.png"}
                },
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.com/image2.png"}
                },
                {
                    "type": "text",
                    "text": "Compare these two images"
                }
            ]
        }
    ]
)
print(response.choices[0].message)
```

### Base64 Image Input

```python
import base64
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

# Read and encode image
with open("image.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/png;base64,{image_data}"
                    }
                },
                {
                    "type": "text",
                    "text": "Describe this image"
                }
            ]
        }
    ]
)
print(response.choices[0].message)
```

## Video Understanding

GLM-4.6V can also process video content:

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "video_url",
                    "video_url": {"url": "https://example.com/video.mp4"}
                },
                {
                    "type": "text",
                    "text": "Summarize what happens in this video"
                }
            ]
        }
    ]
)
print(response.choices[0].message)
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
