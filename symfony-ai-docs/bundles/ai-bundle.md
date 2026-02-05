# Symfony AI Bundle Documentation

## Overview

The AI Bundle is Symfony's official integration bundle for AI components, integrating:
- Symfony AI Agent
- Symfony AI Chat
- Symfony AI Platform
- Symfony AI Store

## Installation

```bash
composer require symfony/ai-bundle
```

## Configuration

### Basic Example with OpenAI

```yaml
# config/packages/ai.yaml
ai:
    platform:
        openai:
            api_key: '%env(OPENAI_API_KEY)%'
    agent:
        default:
            model: 'gpt-4o-mini'
```

### Advanced Example with Multiple Agents

```yaml
ai:
    platform:
        anthropic:
            api_key: '%env(ANTHROPIC_API_KEY)%'
        azure:
            gpt_deployment:
                base_url: '%env(AZURE_OPENAI_BASEURL)%'
                deployment: '%env(AZURE_OPENAI_GPT)%'
                api_key: '%env(AZURE_OPENAI_KEY)%'
                api_version: '%env(AZURE_GPT_VERSION)%'
        bedrock:
            default: ~
            eu:
                bedrock_runtime_client: 'async_aws.client.bedrock_runtime_eu'
        eleven_labs:
            host: '%env(ELEVEN_LABS_HOST)%'
            api_key: '%env(ELEVEN_LABS_API_KEY)%'
            output_path: '%env(ELEVEN_LABS_OUTPUT_PATH)%'
        gemini:
            api_key: '%env(GEMINI_API_KEY)%'
        perplexity:
            api_key: '%env(PERPLEXITY_API_KEY)%'
        vertexai:
            location: '%env(GOOGLE_CLOUD_LOCATION)%'
            project_id: '%env(GOOGLE_CLOUD_PROJECT)%'
            api_key: '%env(GOOGLE_CLOUD_VERTEX_API_KEY)%'
        ollama:
            host_url: '%env(OLLAMA_HOST_URL)%'
        transformersphp: ~
    agent:
        rag:
            platform: 'ai.platform.azure.gpt_deployment'
            model: 'gpt-4o-mini'
            memory: 'You have access to conversation history and user preferences'
            prompt:
                text: 'You are a helpful assistant that can answer questions.'
                include_tools: true
            tools:
                - 'Symfony\AI\Agent\Bridge\SimilaritySearch\SimilaritySearch'
                - service: 'App\Agent\Tool\CompanyName'
                  name: 'company_name'
                  description: 'Provides the name of your company'
                  method: 'foo'
                - agent: 'research'
                  name: 'wikipedia_research'
                  description: 'Can research on Wikipedia'
        research:
            platform: 'ai.platform.anthropic'
            model: 'claude-3-7-sonnet'
            tools:
                - 'Symfony\AI\Agent\Bridge\Wikipedia\Wikipedia'
            fault_tolerant_toolbox: false
```

## Platforms

### Generic Platform

For services complying with OpenAI API (e.g., LiteLLM):

```yaml
ai:
    platform:
        generic:
            litellm:
                base_url: '%env(LITELLM_HOST_URL)%'
                api_key: '%env(LITELLM_API_KEY)%'
                model_catalog: 'Symfony\AI\Platform\Bridge\Generic\ModelCatalog'
    agent:
        test:
            platform: 'ai.platform.generic.litellm'
            model: 'mistral-small-latest'
            tools: false

services:
    Symfony\AI\Platform\Bridge\Generic\ModelCatalog:
        $models:
            mistral-small-latest:
                class: 'Symfony\AI\Platform\Bridge\Generic\CompletionsModel'
                capabilities:
                    - !php/const 'Symfony\AI\Platform\Capability::INPUT_MESSAGES'
                    - !php/const 'Symfony\AI\Platform\Capability::OUTPUT_TEXT'
                    - !php/const 'Symfony\AI\Platform\Capability::OUTPUT_STREAMING'
                    - !php/const 'Symfony\AI\Platform\Capability::OUTPUT_STRUCTURED'
                    - !php/const 'Symfony\AI\Platform\Capability::INPUT_IMAGE'
                    - !php/const 'Symfony\AI\Platform\Capability::TOOL_CALLING'
```

### Cached Platform

```yaml
ai:
    platform:
        openai:
            api_key: '%env(OPENAI_API_KEY)%'
        cache:
            openai:
                platform: 'ai.platform.openai'
                service: 'cache.app'
    agent:
        openai:
            platform: 'ai.platform.cache.openai'
            model: 'gpt-4o-mini'
```

## Store Dependency Injection

```yaml
ai:
    store:
        memory:
            main:
                strategy: 'cosine'
            products:
                strategy: 'manhattan'
        chromadb:
            main:
                collection: 'documents'
```

Automatically registered aliases:
- `StoreInterface $main` - References memory store (first occurrence)
- `StoreInterface $memoryMain` - Explicitly references memory store
- `StoreInterface $chromadbMain` - Explicitly references chromadb store
- `StoreInterface $products` - References memory products store

```php
use Symfony\AI\Store\StoreInterface;

final readonly class DocumentService
{
    public function __construct(
        private StoreInterface $main,              // Uses memory store
        private StoreInterface $chromadbMain,      // Explicitly uses chromadb
        private StoreInterface $memoryProducts,    // Explicitly uses memory products
    ) {
    }
}
```

## Model Configuration

### Query Parameters Style

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini?temperature=0.7&max_output_tokens=2000&stream=true'
```

### Options Style

```yaml
ai:
    agent:
        my_agent:
            model:
                name: 'gpt-4o-mini'
                options:
                    temperature: 0.7
                    max_output_tokens: 2000
                    stream: true
```

## HTTP Client Configuration

```yaml
ai:
    platform:
        openai:
            api_key: '%env(OPENAI_API_KEY)%'
            http_client: 'app.custom_http_client'
```

## System Prompt Configuration

### Basic String

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            prompt: 'You are a helpful assistant.'
```

### Advanced Array Format

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            prompt:
                text: 'You are a helpful assistant that can answer questions.'
                include_tools: true
                enable_translation: true
                translation_domain: 'ai_prompts'
```

### File-Based Prompts

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            prompt:
                file: '%kernel.project_dir%/prompts/assistant.txt'
```

**prompts/assistant.txt**:
```
You are a helpful and knowledgeable assistant.

Guidelines:
- Be clear and direct in your responses
- Provide examples when appropriate
- Be respectful and professional at all times
```

### Translation Support

Requires: `composer require symfony/translation`

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            prompt:
                text: 'agent.system_prompt'
                enable_translation: true
                translation_domain: 'ai_prompts'
```

## Memory Provider Configuration

### Static Memory

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            memory: 'You have access to user preferences and conversation history'
            prompt:
                text: 'You are a helpful assistant.'
```

### Dynamic Memory

```yaml
ai:
    agent:
        my_agent:
            model: 'gpt-4o-mini'
            memory:
                service: 'my_memory_service'
            prompt:
                text: 'You are a helpful assistant.'
```

### Custom Memory Provider

```php
use Symfony\AI\Agent\Input;
use Symfony\AI\Agent\Memory\Memory;
use Symfony\AI\Agent\Memory\MemoryProviderInterface;

final class MyMemoryProvider implements MemoryProviderInterface
{
    public function loadMemory(Input $input): array
    {
        return [
            new Memory('Username: OskarStark'),
            new Memory('Age: 40'),
            new Memory('User preferences: prefers concise answers'),
        ];
    }
}
```

## Multi-Agent Orchestration

```yaml
ai:
    multi_agent:
        support:
            orchestrator: 'orchestrator'
            handoffs:
                technical: ['bug', 'problem', 'technical', 'error', 'code', 'debug']
            fallback: 'general'
```

Usage:

```php
use Symfony\AI\Agent\AgentInterface;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

final class SupportController
{
    public function __construct(
        #[Autowire(service: 'ai.multi_agent.support')]
        private AgentInterface $supportAgent,
    ) {
    }

    public function askSupport(string $question): string
    {
        $messages = new MessageBag(Message::ofUser($question));
        $response = $this->supportAgent->call($messages);

        return $response->getContent();
    }
}
```

## Console Commands

### `ai:platform:invoke`

```bash
php bin/console ai:platform:invoke openai gpt-4o-mini "Hello, world!"
php bin/console ai:platform:invoke anthropic claude-3-5-sonnet-20241022 "Explain quantum physics"
```

### `ai:agent:call`

```bash
php bin/console ai:agent:call default
php bin/console ai:agent:call wikipedia
```

### `ai:store:setup`

```bash
php bin/console ai:store:setup chromadb.default
```

### `ai:store:drop`

```bash
php bin/console ai:store:drop chromadb.default --force
```

### `ai:store:index`

```bash
php bin/console ai:store:index default
php bin/console ai:store:index blog --source=/path/to/file.txt
php bin/console ai:store:index blog --source=/path/to/file1.txt --source=/path/to/file2.txt
```

## Usage

### Agent Service

```php
use Symfony\AI\Agent\AgentInterface;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

final readonly class MyService
{
    public function __construct(
        private AgentInterface $agent,
    ) {
    }

    public function submit(string $message): string
    {
        $messages = new MessageBag(
            Message::forSystem('Speak like a pirate.'),
            Message::ofUser($message),
        );

        return $this->agent->call($messages);
    }
}
```

### Register Processors

```php
use Symfony\AI\Agent\Input;
use Symfony\AI\Agent\InputProcessorInterface;
use Symfony\AI\Agent\Output;
use Symfony\AI\Agent\OutputProcessorInterface;
use Symfony\AI\Agent\Attribute\AsInputProcessor;
use Symfony\AI\Agent\Attribute\AsOutputProcessor;

#[AsInputProcessor(priority: 99)]
#[AsOutputProcessor(agent: 'ai.agent.my_agent_name')]
final readonly class MyService implements InputProcessorInterface, OutputProcessorInterface
{
    public function processInput(Input $input): void
    {
        // Process input
    }

    public function processOutput(Output $output): void
    {
        // Process output
    }
}
```

### Register Tools

Available tool packages:
```bash
composer require symfony/ai-brave-tool
composer require symfony/ai-clock-tool
composer require symfony/ai-firecrawl-tool
composer require symfony/ai-mapbox-tool
composer require symfony/ai-open-meteo-tool
composer require symfony/ai-scraper-tool
composer require symfony/ai-serp-api-tool
composer require symfony/ai-similarity-search-tool
composer require symfony/ai-tavily-tool
composer require symfony/ai-wikipedia-tool
composer require symfony/ai-youtube-tool
```

## Creating Custom Tools

```php
use Symfony\AI\Agent\Toolbox\Attribute\AsTool;

#[AsTool('company_name', 'Provides the name of your company')]
final class CompanyName
{
    public function __invoke(): string
    {
        return 'ACME Corp.';
    }
}
```

Disable all tools:
```yaml
ai:
    agent:
        my_agent:
            tools: false
```

Specify specific tools:
```yaml
ai:
    agent:
        my_agent:
            tools:
                - 'Symfony\AI\Agent\Bridge\SimilaritySearch\SimilaritySearch'
```

### Tool Access Control

Requires: `symfony/security-core`

```php
use Symfony\AI\AiBundle\Security\Attribute\IsGrantedTool;
use Symfony\AI\Agent\Toolbox\Attribute\AsTool;

#[IsGrantedTool('ROLE_ADMIN')]
#[AsTool('company_name', 'Provides the name of your company')]
final class CompanyName
{
    public function __invoke(): string
    {
        return 'ACME Corp.';
    }
}
```

## Token Usage Tracking

```php
use Symfony\AI\Agent\AgentInterface;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;
use Symfony\AI\Platform\Result\Metadata\TokenUsage\TokenUsage;

final readonly class MyService
{
    public function __construct(
        private AgentInterface $agent,
    ) {
    }

    public function getTokenUsage(string $message): ?TokenUsage
    {
        $messages = new MessageBag(Message::ofUser($message));
        $result = $this->agent->call($messages);

        return $result->getMetadata()->get('token_usage');
    }
}
```

## Vectorizers

### Configuration

```yaml
ai:
    vectorizer:
        openai_small:
            platform: 'ai.platform.openai'
            model:
                name: 'text-embedding-3-small'
                options:
                    dimensions: 512

        openai_large:
            platform: 'ai.platform.openai'
            model: 'text-embedding-3-large'

        mistral_embed:
            platform: 'ai.platform.mistral'
            model: 'mistral-embed'
```

### Using in Indexers

```yaml
ai:
    indexer:
        documents:
            loader: 'Symfony\AI\Store\Document\Loader\TextFileLoader'
            vectorizer: 'ai.vectorizer.openai_small'
            store: 'ai.store.chromadb.documents'

        research:
            loader: 'Symfony\AI\Store\Document\Loader\TextFileLoader'
            vectorizer: 'ai.vectorizer.openai_large'
            store: 'ai.store.chromadb.research'
```

## Retrievers

### Configuration

```yaml
ai:
    retriever:
        default:
            vectorizer: 'ai.vectorizer.openai_small'
            store: 'ai.store.chromadb.default'

        research:
            vectorizer: 'ai.vectorizer.mistral_embed'
            store: 'ai.store.memory.research'
```

### Usage

```php
use Symfony\AI\Store\RetrieverInterface;

final readonly class MyService
{
    public function __construct(
        private RetrieverInterface $retriever,
    ) {
    }

    public function search(string $query): array
    {
        $documents = [];
        foreach ($this->retriever->retrieve($query) as $document) {
            $documents[] = $document;
        }

        return $documents;
    }
}
```

### Injecting Specific Retriever

```php
use Symfony\AI\Store\RetrieverInterface;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

final readonly class ResearchService
{
    public function __construct(
        #[Autowire(service: 'ai.retriever.research')]
        private RetrieverInterface $retriever,
    ) {
    }
}
```

## Message Stores

### Configuration

```yaml
ai:
    message_store:
        cache:
            youtube:
                service: 'cache.app'
                key: 'youtube'
```

## Chats

### Configuration

```yaml
ai:
    chat:
        youtube:
            agent: 'ai.agent.youtube'
            message_store: 'ai.message_store.cache.youtube'
```

---

**License**: This work is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.
