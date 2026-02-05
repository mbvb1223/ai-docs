# Z.AI Python SDK

The Z.AI Python SDK is an official toolkit enabling Python developers to integrate AI models efficiently with Pythonic API design.

## Core Capabilities

- Chat conversations (single/multi-turn, streaming)
- Function calling for custom operations
- Vision understanding and image analysis
- Image and video generation from text
- Speech processing (text-to-speech, speech-to-text)
- Text embedding and semantic search
- Intelligent assistant development
- Content moderation tools

## Installation & Setup

### Requirements

- Python 3.8 or higher
- Valid Z.AI API key
- Package manager (pip, poetry, or uv)

### Install via pip

```bash
pip install zai-sdk
```

### Verify installation

```python
import zai
print(zai.__version__)
```

### Initialize client

```python
from zai import ZaiClient

# Direct API key
client = ZaiClient(api_key="your-api-key")

# Or use environment variable
# export ZAI_API_KEY=your-api-key
client = ZaiClient()
```

## Basic Usage

### Simple Chat

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(response.choices[0].message.content)
```

### Streaming Responses

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[{"role": "user", "content": "Tell a story"}],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### Multi-turn Conversations

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "user", "content": "What is Python?"},
        {"role": "assistant", "content": "Python is a programming language..."},
        {"role": "user", "content": "What are its main features?"}
    ]
)

print(response.choices[0].message.content)
```

### System Messages

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": "How do I read a file in Python?"}
    ]
)

print(response.choices[0].message.content)
```

## Advanced Features

### Thinking Mode

Enable reasoning for complex tasks:

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[{"role": "user", "content": "Solve this math problem: 15 * 23 + 47"}],
    thinking={"type": "enabled"},
    max_tokens=4096,
    temperature=1.0
)

print(response.choices[0].message)
```

### Function Calling

Define and use custom functions:

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)

# Check if model wants to call a function
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    print(f"Function: {tool_call.function.name}")
    print(f"Arguments: {tool_call.function.arguments}")
```

### Vision (GLM-4.6V)

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.6v",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}},
                {"type": "text", "text": "Describe this image"}
            ]
        }
    ]
)

print(response.choices[0].message.content)
```

### Image Generation

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.images.generations(
    model="glm-image",
    prompt="A sunset over mountains",
    size="1280x1280"
)

print(response.data[0].url)
```

### Video Generation

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.videos.generations(
    model="cogvideox-3",
    prompt="A bird flying over the ocean",
    quality="quality",
    size="1920x1080",
    fps=30
)

# Retrieve result
result = client.videos.retrieve_videos_result(id=response.id)
print(result.video_url)
```

## Error Handling

```python
from zai import ZaiClient
import zai

client = ZaiClient(api_key="your-api-key")

try:
    response = client.chat.completions.create(
        model="glm-4.7",
        messages=[{"role": "user", "content": "Hello"}]
    )
except zai.core.APIStatusError as err:
    print(f"API error: {err.status_code} - {err.message}")
except zai.core.APITimeoutError as err:
    print(f"Request timed out: {err}")
except zai.core.APIConnectionError as err:
    print(f"Connection failed: {err}")
```

## Configuration Options

### Custom Timeout

```python
from zai import ZaiClient

client = ZaiClient(
    api_key="your-api-key",
    timeout=60.0  # 60 seconds
)
```

### Custom Base URL

```python
from zai import ZaiClient

client = ZaiClient(
    api_key="your-api-key",
    base_url="https://api.z.ai/api/coding/paas/v4"  # Coding endpoint
)
```

### Max Retries

```python
from zai import ZaiClient

client = ZaiClient(
    api_key="your-api-key",
    max_retries=3
)
```

## Resources

- **GitHub**: https://github.com/zai-org/z-ai-sdk-python
- **Examples**: https://github.com/zai-org/z-ai-sdk-python/tree/main/examples
- **API Reference**: https://docs.z.ai/api-reference
