# Z.AI Java SDK

Z.AI provides an official Java development toolkit for integrating AI models with enterprise-grade design supporting high concurrency and high availability.

## Core Capabilities

- Chat conversations (single/multi-turn, streaming/non-streaming)
- Function calling for custom operations
- Vision understanding and image analysis
- Image and video generation from text
- Speech processing (STT/TTS)
- Text embedding for semantic search
- AI assistant development

## Technical Requirements

- Java 1.8 or higher (supports Java 8, 11, 17, 21)
- Maven 3.6+ or Gradle 6.0+
- HTTPS connectivity
- Valid Z.AI API key

## Installation

### Maven

```xml
<dependency>
    <groupId>ai.z.openapi</groupId>
    <artifactId>zai-sdk</artifactId>
    <version>0.3.0</version>
</dependency>
```

### Gradle

```gradle
implementation 'ai.z.openapi:zai-sdk:0.3.0'
```

## Quick Start

### Setup

1. Obtain API key from [Z.AI Open Platform](https://z.ai)
2. Set as environment variable:
   ```bash
   export ZAI_API_KEY=your-api-key
   ```
3. Initialize client

### Initialize Client

```java
import ai.z.openapi.ZaiClient;

ZaiClient client = ZaiClient.builder()
    .ofZAI()
    .apiKey(System.getenv("ZAI_API_KEY"))
    .build();
```

### Basic Chat

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.Arrays;

public class BasicChat {
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
                    .content("Hello, please introduce yourself")
                    .build()
            ))
            .build();

        ChatCompletionResponse response = client.chat().createChatCompletion(request);

        if (response.isSuccess()) {
            String content = response.getData().getChoices().get(0).getMessage().getContent();
            System.out.println(content);
        } else {
            System.err.println("Error: " + response.getMsg());
        }
    }
}
```

### Multi-turn Conversation

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.Arrays;

public class MultiTurnChat {
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
                    .content("What is Java?")
                    .build(),
                ChatMessage.builder()
                    .role(ChatMessageRole.ASSISTANT.value())
                    .content("Java is a programming language...")
                    .build(),
                ChatMessage.builder()
                    .role(ChatMessageRole.USER.value())
                    .content("What are its main features?")
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

## Streaming Responses

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.Arrays;

public class StreamingChat {
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
                    .content("Tell me a story")
                    .build()
            ))
            .stream(true)
            .build();

        ChatCompletionResponse response = client.chat().createChatCompletionStream(request);

        response.getFlowable().subscribe(
            data -> {
                Delta content = data.getChoices().get(0).getDelta();
                if (content != null && content.getContent() != null) {
                    System.out.print(content.getContent());
                }
            },
            error -> System.err.println("Error: " + error.getMessage()),
            () -> System.out.println("\nComplete")
        );
    }
}
```

## Thinking Mode

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.Arrays;

public class ThinkingMode {
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
                    .content("Solve: What is 15 * 23 + 47?")
                    .build()
            ))
            .thinking(ChatThinking.builder().type("enabled").build())
            .maxTokens(4096)
            .temperature(1.0f)
            .build();

        ChatCompletionResponse response = client.chat().createChatCompletion(request);

        if (response.isSuccess()) {
            System.out.println(response.getData().getChoices().get(0).getMessage());
        }
    }
}
```

## Function Calling

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import java.util.*;

public class FunctionCalling {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder()
            .ofZAI()
            .apiKey("your-api-key")
            .build();

        // Define function parameters
        Map<String, Object> properties = new HashMap<>();
        properties.put("location", Map.of(
            "type", "string",
            "description", "City name"
        ));

        ChatTool weatherTool = ChatTool.builder()
            .type(ChatToolType.FUNCTION.value())
            .function(ChatFunction.builder()
                .name("get_weather")
                .description("Get weather for specified location")
                .parameters(ChatFunctionParameters.builder()
                    .type("object")
                    .properties(properties)
                    .required(Arrays.asList("location"))
                    .build())
                .build())
            .build();

        ChatCompletionCreateParams request = ChatCompletionCreateParams.builder()
            .model("glm-4.7")
            .messages(Arrays.asList(
                ChatMessage.builder()
                    .role(ChatMessageRole.USER.value())
                    .content("What's the weather in Tokyo?")
                    .build()
            ))
            .tools(Arrays.asList(weatherTool))
            .toolChoice("auto")
            .build();

        ChatCompletionResponse response = client.chat().createChatCompletion(request);

        if (response.isSuccess()) {
            ChatMessage message = response.getData().getChoices().get(0).getMessage();
            if (message.getToolCalls() != null && !message.getToolCalls().isEmpty()) {
                ToolCall toolCall = message.getToolCalls().get(0);
                System.out.println("Function: " + toolCall.getFunction().getName());
                System.out.println("Arguments: " + toolCall.getFunction().getArguments());
            }
        }
    }
}
```

## Image Generation

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.image.*;

public class ImageGeneration {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder()
            .ofZAI()
            .apiKey("your-api-key")
            .build();

        CreateImageRequest request = CreateImageRequest.builder()
            .model("glm-image")
            .prompt("A sunset over mountains")
            .size("1280x1280")
            .build();

        ImageResponse response = client.images().createImage(request);
        System.out.println(response.getData().get(0).getUrl());
    }
}
```

## Error Handling

```java
import ai.z.openapi.ZaiClient;
import ai.z.openapi.service.model.*;
import ai.z.openapi.core.exception.*;

public class ErrorHandling {
    public static void main(String[] args) {
        ZaiClient client = ZaiClient.builder()
            .ofZAI()
            .apiKey("your-api-key")
            .build();

        try {
            ChatCompletionCreateParams request = ChatCompletionCreateParams.builder()
                .model("glm-4.7")
                .messages(Arrays.asList(
                    ChatMessage.builder()
                        .role(ChatMessageRole.USER.value())
                        .content("Hello")
                        .build()
                ))
                .build();

            ChatCompletionResponse response = client.chat().createChatCompletion(request);

            if (!response.isSuccess()) {
                System.err.println("API Error: " + response.getCode() + " - " + response.getMsg());
            }
        } catch (ApiException e) {
            System.err.println("API Exception: " + e.getMessage());
        } catch (Exception e) {
            System.err.println("Unexpected error: " + e.getMessage());
        }
    }
}
```

## Resources

- **GitHub**: z-ai-sdk-java repository
- **API Reference**: https://docs.z.ai/api-reference
- **Sample Projects**: Repository samples directory
