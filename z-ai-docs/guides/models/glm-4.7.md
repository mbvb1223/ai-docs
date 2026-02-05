# GLM-4.7

GLM-4.7 Series represents Z.AI's latest flagship models with enhanced programming capabilities and improved multi-step reasoning.

## Model Variants

| Model | Positioning | Context | Max Output |
|-------|-------------|---------|-----------|
| GLM-4.7 | Flagship, highest performance | 200K | 128K tokens |
| GLM-4.7-FlashX | Lightweight, high-speed, affordable | 200K | 128K tokens |
| GLM-4.7-Flash | Lightweight, completely free | 200K | 128K tokens |

## Key Capabilities

- **Thinking Mode**: Multiple reasoning modes for different scenarios
- **Streaming Output**: Real-time response streaming
- **Function Calling**: Tool invocation integration
- **Context Caching**: Performance optimization for long conversations
- **Structured Output**: JSON and similar formats

## Primary Use Cases

1. **Agentic Coding** - Task completion rather than single-point generation, producing executable frameworks
2. **Real-time Applications** - System-level integration of visual recognition and logic control
3. **Web UI Generation** - Enhanced visual code understanding and aesthetic output
4. **Collaborative Problem-Solving** - Reliable context maintenance across multi-turn dialogue
5. **Creative Writing** - Nuanced prose with consistent character and world-building
6. **Office Creation** - PPT and poster generation with improved layout consistency
7. **Research Tasks** - Information retrieval with structured organization

## Performance Highlights

- Achieves top open-source ranking on SWE-bench-Verified
- Surpasses Claude Sonnet 4.5 on LiveCodeBench V6 with 84.9 points

## Quick Start Examples

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
        {
            "role": "user",
            "content": "As a marketing expert, please create an attractive slogan for my product.",
        },
        {
            "role": "assistant",
            "content": "Sure, to craft a compelling slogan, please tell me more about your product.",
        },
        {"role": "user", "content": "Z.AI Open Platform"},
    ],
    thinking={"type": "enabled"},
    max_tokens=4096,
    temperature=1.0,
)

print(response.choices[0].message)
```

### Streaming Example

```python
from zai import ZaiClient

client = ZaiClient(api_key="your-api-key")

response = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {
            "role": "user",
            "content": "As a marketing expert, please create an attractive slogan for my product.",
        },
        {
            "role": "assistant",
            "content": "Sure, to craft a compelling slogan, please tell me more about your product.",
        },
        {"role": "user", "content": "Z.AI Open Platform"},
    ],
    thinking={"type": "enabled"},
    stream=True,
    max_tokens=4096,
    temperature=0.6,
)

for chunk in response:
    if chunk.choices[0].delta.reasoning_content:
        print(chunk.choices[0].delta.reasoning_content, end="", flush=True)
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### cURL Example

```bash
curl -X POST "https://api.z.ai/api/paas/v4/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "glm-4.7",
    "messages": [
      {
        "role": "user",
        "content": "As a marketing expert, please create an attractive slogan for my product."
      },
      {
        "role": "assistant",
        "content": "Sure, to craft a compelling slogan, please tell me more about your product."
      },
      {
        "role": "user",
        "content": "Z.AI Open Platform"
      }
    ],
    "thinking": {
      "type": "enabled"
    },
    "max_tokens": 4096,
    "temperature": 1.0
  }'
```

### Java Example

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.ChatCompletionCreateParams;
import ai.z.openapi.service.model.ChatCompletionResponse;
import ai.z.openapi.service.model.ChatMessage;
import ai.z.openapi.service.model.ChatMessageRole;
import ai.z.openapi.service.model.ChatThinking;
import java.util.Arrays;

public class BasicChat {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder().ofZAI().apiKey("your-api-key").build();

        ChatCompletionCreateParams request =
                ChatCompletionCreateParams.builder()
                        .model("glm-4.7")
                        .messages(
                                Arrays.asList(
                                        ChatMessage.builder()
                                                .role(ChatMessageRole.USER.value())
                                                .content(
                                                        "As a marketing expert, please create an attractive slogan for my product.")
                                                .build(),
                                        ChatMessage.builder()
                                                .role(ChatMessageRole.ASSISTANT.value())
                                                .content(
                                                        "Sure, to craft a compelling slogan, please tell me more about your product.")
                                                .build(),
                                        ChatMessage.builder()
                                                .role(ChatMessageRole.USER.value())
                                                .content("Z.AI Open Platform")
                                                .build()))
                        .thinking(ChatThinking.builder().type("enabled").build())
                        .maxTokens(4096)
                        .temperature(1.0f)
                        .build();

        ChatCompletionResponse response = client.chat().createChatCompletion(request);

        if (response.isSuccess()) {
            Object reply = response.getData().getChoices().get(0).getMessage();
            System.out.println("AI Response: " + reply);
        } else {
            System.err.println("Error: " + response.getMsg());
        }
    }
}
```

### OpenAI Python SDK Compatibility

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-Z.AI-api-key",
    base_url="https://api.z.ai/api/paas/v4/",
)

completion = client.chat.completions.create(
    model="glm-4.7",
    messages=[
        {"role": "system", "content": "You are a smart and creative novelist"},
        {
            "role": "user",
            "content": "Please write a short fairy tale story as a fairy tale master",
        },
    ],
)

print(completion.choices[0].message.content)
```

## Installation

**Python Official SDK:**
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

**Java (Gradle):**
```groovy
implementation 'ai.z.openapi:zai-sdk:0.3.0'
```

**OpenAI SDK:**
```bash
pip install --upgrade 'openai>=1.0'
```

## Reasoning Capabilities

The model supports three reasoning approaches:

- **Interleaved Reasoning**: Performs analysis before each response/tool invocation
- **Retention-Based Reasoning**: Preserves reasoning blocks across multi-turn conversations
- **Round-Level Reasoning**: Enables per-session control of reasoning overhead
