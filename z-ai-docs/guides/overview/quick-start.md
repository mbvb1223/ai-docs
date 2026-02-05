# Z.AI Quick Start

This guide helps you get started with Z.AI's AI models quickly.

## Getting Started Steps

### 1. API Key Setup

1. Register or login at [Z.AI Open Platform](https://z.ai)
2. Top up billing if needed
3. Create an API key from the API Keys management page
4. Copy the key for use in requests

### 2. Model Selection

Available models include:

| Model | Description |
|-------|-------------|
| **GLM-4.7** | Latest flagship model with open-source SOTA performance and superior Agentic Coding Experience |
| **GLM-4.6V** | Multimodal model with 128K context window for vision tasks |
| **GLM-Image** | Text-to-image generation capabilities |
| **CogVideoX-3** | Video generation with frame interpolation for improved image stability |

### 3. Development Approach Options

Supported calling methods:
- HTTP API (RESTful)
- Official Z.AI Python SDK
- Official Z.AI Java SDK
- OpenAI-compatible Python SDK
- OpenAI-compatible Node.js SDK
- OpenAI-compatible Java SDK

### 4. Making API Calls

**API Endpoint:**
```
https://api.z.ai/api/paas/v4/chat/completions
```

**Note:** When using the GLM Coding Plan, configure the dedicated Coding endpoint:
```
https://api.z.ai/api/coding/paas/v4
```

## Quick Examples

### cURL

```bash
curl -X POST "https://api.z.ai/api/paas/v4/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "glm-4.7",
    "messages": [
      {"role": "user", "content": "Hello, how are you?"}
    ]
  }'
```

### Python (Official SDK)

```bash
pip install zai-sdk
```

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "user", "content": "Hello, how are you?"}
    ]
)

print(response.choices[0].message.content)
```

### Python (OpenAI Compatible)

```bash
pip install --upgrade 'openai>=1.0'
```

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-Z.AI-api-key",
    base_url="https://api.z.ai/api/paas/v4/",
)

completion = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello!"}
    ]
)

print(completion.choices[0].message.content)
```

### Java

**Maven:**
```xml
<dependency>
    <groupId>ai.z.openapi</groupId>
    <artifactId>zai-sdk</artifactId>
    <version>0.3.0</version>
</dependency>
```

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.Arrays;

public class QuickStart {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder()
            .ofZAI()
            .apiKey("your-api-key")
            .build();

        ChatCompletionCreateParams request = ChatCompletionCreateParams.builder()
            .model("glm-4.7")
            .messages(Arrays.asList(
                ChatMessage.builder()
                    .role(ChatMessageRole.USER.value())
                    .content("Hello, how are you?")
                    .build()
            ))
            .build();

        ChatCompletionResponse response = client.chat().createChatCompletion(request);

        if (response.isSuccess()) {
            System.out.println(response.getData().getChoices().get(0).getMessage().getContent());
        }
    }
}
```

## Next Steps

- Explore [GLM-4.7](../models/glm-4.7.md) for text generation
- Try [GLM-4.6V](../models/glm-4.6v.md) for vision tasks
- Generate images with [GLM-Image](../models/glm-image.md)
- Create videos with [CogVideoX-3](../models/cogvideox-3.md)
