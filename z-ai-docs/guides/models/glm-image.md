# GLM-Image

GLM-Image is Z.AI's flagship image generation model utilizing a hybrid "autoregressive + diffusion decoder" architecture. This approach balances comprehensive instruction comprehension with precise detail rendering.

## Key Specifications

| Aspect | Details |
|--------|---------|
| **Pricing** | $0.015 per image |
| **Input** | Text prompts |
| **Output** | Image URLs (requires download) |
| **Supported Aspect Ratios** | 1:1, 3:4, 4:3, 16:9, and others |
| **Resolution Range** | 512–2048px (multiples of 32) |
| **Recommended Sizes** | 1280×1280, 1568×1056, 1472×1088, 1728×960 |

## Primary Use Cases

- **Commercial Posters**: Festival promotions and brand marketing with strong visual hierarchy
- **Educational Graphics**: Complex diagrams with logical relationships and annotations
- **Multi-Panel Content**: Consistent styling across e-commerce displays and comics
- **Social Media Assets**: Dynamic layouts with sophisticated design elements

## Technical Architecture

The model combines:
- **9B autoregressive component**: Focuses on semantic understanding and composition
- **7B DiT diffusion decoder**: Emphasizes high-frequency details and text rendering accuracy

### Performance Benchmarks

GLM-Image achieves state-of-the-art results among open-source models:
- **CVTG-2K leaderboard**: 0.9116 word accuracy, 0.9557 normalized edit distance
- **LongText-Bench**: 0.9524 English, 0.9788 Chinese performance scores

## Quick Start Examples

### cURL

```bash
curl --request POST \
  --url https://api.z.ai/api/paas/v4/images/generations \
  --header 'Authorization: Bearer <token>' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "glm-image",
    "prompt": "A cute little kitten sitting on a sunny windowsill, with the background of blue sky and white clouds.",
    "size": "1280x1280"
  }'
```

### Python

```bash
pip install zai-sdk
```

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.images.generations(
    model="glm-image",
    prompt="A cute little kitten sitting on a sunny windowsill, with the background of blue sky and white clouds.",
    size="1280x1280"
)

print(response.data[0].url)
```

### Java

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.image.CreateImageRequest;
import ai.z.openapi.service.image.ImageResponse;

public class GlmImageExample {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder()
            .ofZAI()
            .apiKey("YOUR_API_KEY")
            .build();

        CreateImageRequest request = CreateImageRequest.builder()
            .model("glm-image")
            .prompt("A cute little kitten sitting on a sunny windowsill, with the background of blue sky and white clouds.")
            .size("1280x1280")
            .build();

        ImageResponse response = client.images().createImage(request);
        System.out.println(response.getData());
    }
}
```

## Advanced Options

### Custom Dimensions

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

# Landscape format
response = client.images.generations(
    model="glm-image",
    prompt="A panoramic mountain landscape at sunset",
    size="1728x960"
)

# Portrait format
response = client.images.generations(
    model="glm-image",
    prompt="A tall skyscraper reaching into the clouds",
    size="960x1728"
)
```

### Batch Generation

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

prompts = [
    "A red apple on a wooden table",
    "A blue car on a city street",
    "A green forest in morning mist"
]

for prompt in prompts:
    response = client.images.generations(
        model="glm-image",
        prompt=prompt,
        size="1280x1280"
    )
    print(f"Generated: {response.data[0].url}")
```

## Downloading Generated Images

The API returns URLs that must be downloaded separately:

```python
import requests
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.images.generations(
    model="glm-image",
    prompt="A beautiful sunset over the ocean",
    size="1280x1280"
)

# Download the image
image_url = response.data[0].url
image_response = requests.get(image_url)

with open("generated_image.png", "wb") as f:
    f.write(image_response.content)
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
    <version>0.3.2</version>
</dependency>
```

**Java (Gradle):**
```groovy
implementation 'ai.z.openapi:zai-sdk:0.3.2'
```
