# Symfony AI - Platform Component Documentation

## Overview

The Platform component provides a unified abstraction for interacting with different AI models, their providers, and contracts. It allows developers to switch between different AI models and providers without changing application code.

## Installation

```bash
composer require symfony/ai-platform
```

## Purpose

The Platform component enables:
- Unified interface for various AI models from different providers
- Easy switching between AI models and providers
- Flexibility based on specific use cases or performance requirements
- Support for multiple model types and capabilities

## Usage

### Basic Setup

Platform instantiation is typically delegated to provider-specific factories:

```php
use Symfony\AI\Platform\Bridge\OpenAi\Embeddings;
use Symfony\AI\Platform\Bridge\OpenAi\Gpt;
use Symfony\AI\Platform\Bridge\OpenAi\PlatformFactory;

$platform = PlatformFactory::create(env('OPENAI_API_KEY'));
```

### Basic Invocation

```php
// Generate vector embedding
$vectorResult = $platform->invoke($embeddings, 'What is the capital of France?');

// Generate text completion
$result = $platform->invoke('gpt-4o-mini', new MessageBag(
    Message::ofUser('What is the capital of France?')
));
```

## Models

### Model Configuration

Models combine:
- Model name
- Set of capabilities
- Additional options

Capabilities are defined by `Capability` (e.g., `Capability::INPUT_AUDIO`, `Capability::OUTPUT_IMAGE`).

### Model Size Variants

For providers like Ollama, specify variants using colon notation:

```php
use Symfony\AI\Platform\Bridge\Ollama\ModelCatalog;

$catalog = new ModelCatalog();

// Get model with size variant
$model = $catalog->getModel('qwen3:32b');

// Get model with size variant and query parameters
$model = $catalog->getModel('qwen3:32b?temperature=0.5&top_p=0.9');
```

### Custom Models

For custom models in Ollama, use `OllamaApiCatalog`:

```php
use Symfony\AI\Platform\Bridge\Ollama\OllamaApiCatalog;
use Symfony\AI\Platform\Bridge\Ollama\PlatformFactory;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$platform = PlatformFactory::create('http://127.0.0.1:11434', HttpClient::create(),
    new OllamaApiCatalog('http://127.0.0.1:11434', HttpClient::create())
);

$platform->invoke('your_custom_model_name', new MessageBag(
    Message::ofUser(...)
));
```

## Supported Models & Platforms

### Language Models
- **OpenAI's GPT** - OpenAI, Azure, OpenRouter
- **Anthropic's Claude** - Anthropic, AWS Bedrock
- **Meta's Llama** - Azure, Ollama, Replicate, AWS Bedrock, OpenRouter
- **Gemini** - Google, Vertex AI, OpenRouter
- **Vertex AI Gen AI** - Vertex AI
- **DeepSeek's R1** - OpenRouter
- **Amazon's Nova** - AWS Bedrock
- **Mistral** - Mistral, OpenRouter
- **Albert API** - French government's sovereign AI
- **LiteLLM** - Unified platform

### Embeddings Models
- Gemini Text Embeddings
- Vertex AI Text Embeddings
- OpenAI Text Embeddings
- Voyage Embeddings
- Mistral Embed
- Qwen

### Other Models
- OpenAI's Dall·E
- OpenAI's Whisper
- LM Studio & HuggingFace models
- ElevenLabs TTS/STT
- Cartesia TTS/STT
- Decart T2I/T2V

### Generic Platforms

For platforms following OpenAI's API standard:

```php
use Symfony\AI\Platform\Bridge\Generic\PlatformFactory;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$platform = PlatformFactory::create('https://api.example.com', 'sk-xxxxxx', $httpClient, $modelCatalog);

$messages = new MessageBag(
    Message::forSystem('You are a pirate and you write funny.'),
    Message::ofUser('What is the Symfony framework?'),
);
$result = $platform->invoke('model-name', $messages);

echo $result->asText();
```

## Options

Pass model and platform-specific options as the third parameter:

```php
$result = $platform->invoke('gpt-4o-mini', $input, [
    'temperature' => 0.7,
    'max_output_tokens' => 100,
]);
```

## Language Models and Messages

### Message Types and Content

```php
use Symfony\AI\Platform\Message\Content\Image;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messageBag = new MessageBag(
    Message::forSystem('You are a helpful assistant.'),
    Message::ofUser('Please describe this picture?',
        Image::fromFile('/path/to/image.jpg')
    ),
);
```

### Message Unique IDs

Each message automatically receives a UUID v7:

```php
use Symfony\AI\Platform\Message\Message;

$message = Message::ofUser('Hello, AI!');

// Access unique ID
$id = $message->getId(); // UUID instance

// Extract creation timestamp
$createdAt = $id->getDateTime(); // DateTimeImmutable
echo $createdAt->format('Y-m-d H:i:s.u');

// Get string representation
echo $id->toRfc4122();
```

### Message Templates

#### String Templates

```php
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\AI\Platform\Message\Template;

$messages = new MessageBag(
    Message::forSystem(Template::string('You are a {role} assistant.')),
    Message::ofUser('What is PHP?')
);

$result = $platform->invoke('gpt-4o-mini', $messages, [
    'template_vars' => ['role' => 'programming'],
]);
```

#### Expression Templates

```php
$template = Template::expression('price * quantity');
```

#### Template Setup

```php
use Symfony\AI\Platform\EventListener\TemplateRendererListener;
use Symfony\AI\Platform\Message\TemplateRenderer\StringTemplateRenderer;
use Symfony\AI\Platform\Message\TemplateRenderer\TemplateRendererRegistry;
use Symfony\Component\EventDispatcher\EventDispatcher;

$eventDispatcher = new EventDispatcher();
$rendererRegistry = new TemplateRendererRegistry([
    new StringTemplateRenderer(),
]);
$templateListener = new TemplateRendererListener($rendererRegistry);
$eventDispatcher->addSubscriber($templateListener);

$platform = PlatformFactory::create($apiKey, eventDispatcher: $eventDispatcher);
```

## Result Streaming

Stream LLM responses word by word:

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Message\Message;
use Symfony\AI\Message\MessageBag;

$agent = new Agent($model);
$messages = new MessageBag(
    Message::forSystem('You are a thoughtful philosopher.'),
    Message::ofUser('What is the purpose of an ant?'),
);
$result = $agent->call($messages, [
    'stream' => true,
]);

foreach ($result->getContent() as $word) {
    echo $word;
}
```

## Image Processing

```php
use Symfony\AI\Platform\Message\Content\Image;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messages = new MessageBag(
    Message::forSystem('You are an image analyzer bot.'),
    Message::ofUser(
        'Describe the image as a comedian would do it.',
        Image::fromFile('/path/to/image.jpg'),
        Image::fromDataUrl('data:image/png;base64,...'),
        new ImageUrl('https://foo.com/bar.png'),
    ),
);
$result = $agent->call($messages);
```

## Audio Processing

```php
use Symfony\AI\Platform\Message\Content\Audio;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messages = new MessageBag(
    Message::ofUser(
        'What is this recording about?',
        Audio::fromFile('/path/audio.mp3'),
    ),
);
$result = $agent->call($messages);
```

## Embeddings

```php
use Symfony\AI\Platform\Bridge\OpenAi\Embeddings;

// Initialize platform

$vectors = $platform->invoke('text-embedding-3-small', $textInput)->asVectors();

dump($vectors[0]->getData()); // [0.123, -0.456, 0.789, ...]
```

## Structured Output

### PHP Classes as Output

```php
use Symfony\AI\Platform\Bridge\Mistral\PlatformFactory;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\AI\Platform\StructuredOutput\PlatformSubscriber;
use Symfony\Component\EventDispatcher\EventDispatcher;

$dispatcher = new EventDispatcher();
$dispatcher->addSubscriber(new PlatformSubscriber());

$platform = PlatformFactory::create($apiKey, eventDispatcher: $dispatcher);
$messages = new MessageBag(
    Message::forSystem('You are a helpful math tutor.'),
    Message::ofUser('how can I solve 8x + 7 = -23'),
);
$result = $platform->invoke('mistral-small-latest', $messages,
    ['response_format' => MathReasoning::class]
);

dump($result->asObject()); // MathReasoning instance
```

### Array Structures as Output

```php
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messages = new MessageBag(Message::ofUser('What date and time is it?'));
$result = $agent->call($messages, ['response_format' => [
    'type' => 'json_schema',
    'json_schema' => [
        'name' => 'clock',
        'strict' => true,
        'schema' => [
            'type' => 'object',
            'properties' => [
                'date' => ['type' => 'string', 'description' => 'The current date in YYYY-MM-DD format.'],
                'time' => ['type' => 'string', 'description' => 'The current time in HH:MM:SS format.'],
            ],
            'required' => ['date', 'time'],
            'additionalProperties' => false,
        ],
    ],
]]);

dump($result->getContent()); // array
```

## Parallel Platform Calls

```php
// Initialize Platform

foreach ($inputs as $input) {
    $results[] = $platform->invoke('gpt-4o-mini', $input);
}

foreach ($results as $result) {
    echo $result->asText().PHP_EOL;
}
```

## Cached Platform Calls

```php
use Symfony\AI\Platform\Bridge\Cache\CachedPlatform;
use Symfony\AI\Platform\Bridge\OpenAi\PlatformFactory;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\Component\Cache\Adapter\ArrayAdapter;
use Symfony\Component\Cache\Adapter\TagAwareAdapter;
use Symfony\Component\HttpClient\HttpClient;

$platform = PlatformFactory::create($apiKey, HttpClient::create());
$cachedPlatform = new CachedPlatform($platform,
    cache: new TagAwareAdapter(new ArrayAdapter())
);

$firstResult = $cachedPlatform->invoke('gpt-4o-mini',
    new MessageBag(Message::ofUser('What is the capital of France?'))
);

echo $firstResult->getContent().PHP_EOL;

// Second call uses cached result
$secondResult = $cachedPlatform->invoke('gpt-4o-mini',
    new MessageBag(Message::ofUser('What is the capital of France?'))
);

echo $secondResult->getContent().PHP_EOL;
```

## High Availability

Use `FailoverPlatform` for automatic failover to backup providers:

```php
use Symfony\AI\Platform\Bridge\Failover\FailoverPlatform;
use Symfony\AI\Platform\Bridge\Ollama\PlatformFactory as OllamaPlatformFactory;
use Symfony\AI\Platform\Bridge\OpenAi\PlatformFactory as OpenAiPlatformFactory;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\Component\HttpClient\HttpClient;
use Symfony\Component\RateLimiter\RateLimiterFactory;
use Symfony\Component\RateLimiter\Storage\InMemoryStorage;

$rateLimiter = new RateLimiterFactory([
    'policy' => 'sliding_window',
    'id' => 'failover',
    'interval' => '3 seconds',
    'limit' => 1,
], new InMemoryStorage());

$platform = new FailoverPlatform([
    OllamaPlatformFactory::create(env('OLLAMA_HOST_URL'), HttpClient::create()),
    OpenAiPlatformFactory::create(env('OPENAI_API_KEY'), HttpClient::create()),
], $rateLimiter);

$result = $platform->invoke('gpt-4o', new MessageBag(
    Message::forSystem('You are a helpful assistant.'),
    Message::ofUser('Tina has one brother and one sister. How many sisters do Tina\'s siblings have?'),
));

echo $result->asText().PHP_EOL;
```

### Bundle Configuration

```yaml
# config/packages/ai.yaml
ai:
    platform:
        openai:
            # ...
        ollama:
            # ...
        failover:
            ollama_to_openai:
                platforms:
                    - 'ai.platform.ollama'
                    - 'ai.platform.openai'
                rate_limiter: 'limiter.failover_platform'

# config/packages/rate_limiter.yaml
framework:
    rate_limiter:
        failover_platform:
            policy: 'sliding_window'
            limit: 100
            interval: '60 minutes'
```

## Testing Tools

Use `InMemoryPlatform` for unit/integration testing without external API calls:

### Fixed Result

```php
use Symfony\AI\Platform\Test\InMemoryPlatform;

$platform = new InMemoryPlatform('Fake result');
$result = $platform->invoke('gpt-4o-mini', 'What is the capital of France?');

echo $result->asText(); // "Fake result"
```

### Dynamic Text Results

```php
$platform = new InMemoryPlatform(
    fn($model, $input, $options) => "Echo: {$input}"
);

$result = $platform->invoke('gpt-4o-mini', 'Hello AI');
echo $result->asText(); // "Echo: Hello AI"
```

### Vector Results

```php
use Symfony\AI\Platform\Result\VectorResult;
use Symfony\AI\Store\Vector;

$platform = new InMemoryPlatform(
    fn() => new VectorResult(new Vector([0.1, 0.2, 0.3, 0.4]))
);

$result = $platform->invoke('gpt-4o-mini', 'vectorize this text');
$vectors = $result->asVectors(); // Vector with [0.1, 0.2, 0.3, 0.4]
```

### Binary Results

```php
use Symfony\AI\Platform\Result\BinaryResult;

$platform = new InMemoryPlatform(
    fn() => new BinaryResult('fake-pdf-content', 'application/pdf')
);

$result = $platform->invoke('gpt-4o-mini', 'generate PDF document');
$binary = $result->asBinary(); // Binary object with content and MIME type
```

## Server Tools

Some platforms provide built-in server-side tools:
- **Gemini Server Tools**: URL Context, Google Search, Code Execution
- **Vertex AI Server Tools**: URL Context, Google Search, Code Execution

---

**License**: This work is licensed under a [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) license.
