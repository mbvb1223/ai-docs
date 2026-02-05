# Z.AI API Reference

## API Endpoints

**General API Endpoint:**
```
https://api.z.ai/api/paas/v4
```

**Dedicated Coding Endpoint:**
```
https://api.z.ai/api/coding/paas/v4
```

The coding endpoint applies exclusively to coding scenarios and should not be used for general API operations.

## Authentication

The platform uses standard HTTP Bearer authentication requiring an API key obtainable from the API Keys management page.

**Header Format:**
```
Authorization: Bearer ZAI_API_KEY
```

## Available Endpoints

### Chat Completion

**Endpoint:** `POST /chat/completions`

Create a chat completion with the specified model.

**Request:**
```json
{
  "model": "glm-4.7",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant"},
    {"role": "user", "content": "Hello!"}
  ],
  "temperature": 0.7,
  "max_tokens": 1024,
  "stream": false
}
```

**Response:**
```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "glm-4.7",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  }
}
```

### Image Generation

**Endpoint:** `POST /images/generations`

Generate images from text prompts.

**Request:**
```json
{
  "model": "glm-image",
  "prompt": "A sunset over mountains",
  "size": "1280x1280"
}
```

**Response:**
```json
{
  "created": 1234567890,
  "data": [
    {
      "url": "https://..."
    }
  ]
}
```

### Video Generation

**Endpoint:** `POST /videos/generations`

Generate videos from text or images (async).

**Request:**
```json
{
  "model": "cogvideox-3",
  "prompt": "A bird flying over the ocean",
  "quality": "quality",
  "size": "1920x1080",
  "fps": 30,
  "with_audio": true
}
```

**Response:**
```json
{
  "id": "video-xxx",
  "status": "processing"
}
```

### Video Result Retrieval

**Endpoint:** `GET /videos/{id}`

Retrieve the result of a video generation task.

**Response:**
```json
{
  "id": "video-xxx",
  "status": "completed",
  "video_url": "https://..."
}
```

## Models

### LLM Models

| Model | Context | Max Output | Description |
|-------|---------|------------|-------------|
| glm-4.7 | 200K | 128K | Flagship model |
| glm-4.7-flashx | 200K | 128K | High-speed, affordable |
| glm-4.7-flash | 200K | 128K | Free tier |

### Vision Models

| Model | Context | Description |
|-------|---------|-------------|
| glm-4.6v | 128K | Flagship multimodal |
| glm-4.6v-flashx | 128K | High-speed, affordable |
| glm-4.6v-flash | 128K | Free tier |

### Image Models

| Model | Price | Description |
|-------|-------|-------------|
| glm-image | $0.015/image | Text-to-image generation |

### Video Models

| Model | Price | Description |
|-------|-------|-------------|
| cogvideox-3 | $0.2/video | Video generation |

## Request Parameters

### Chat Completion Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| model | string | Yes | Model ID to use |
| messages | array | Yes | List of messages |
| temperature | float | No | Sampling temperature (0-2) |
| max_tokens | int | No | Maximum tokens to generate |
| stream | boolean | No | Enable streaming |
| thinking | object | No | Enable reasoning mode |
| tools | array | No | Available functions |
| tool_choice | string | No | Function calling behavior |

### Message Format

```json
{
  "role": "user|assistant|system",
  "content": "Message text or content array"
}
```

### Multimodal Content

```json
{
  "role": "user",
  "content": [
    {"type": "image_url", "image_url": {"url": "https://..."}},
    {"type": "text", "text": "Describe this image"}
  ]
}
```

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Bad request - invalid parameters |
| 401 | Unauthorized - invalid API key |
| 403 | Forbidden - access denied |
| 404 | Not found - invalid endpoint |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

## Rate Limits

Rate limits vary by plan. Implement exponential backoff when receiving 429 responses.

## SDK Options

- Official Python SDK (`zai-sdk`)
- Official Java SDK
- OpenAI-compatible Python SDK
- OpenAI-compatible Node.js SDK
- OpenAI-compatible Java SDK

## Interactive Playground

An API Playground is available at [docs.z.ai](https://docs.z.ai) for testing API calls directly through the documentation interface.
