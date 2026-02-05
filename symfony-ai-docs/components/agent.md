# Symfony AI - Agent Component Documentation

## Overview

The Agent component provides a framework for building AI agents that interact with users, perform tasks, and manage workflows. It sits on top of the Platform and Store components.

## Installation

```bash
composer require symfony/ai-agent
```

## Basic Usage

### Creating an Agent

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Platform\Bridge\OpenAi\PlatformFactory;

$platform = PlatformFactory::create($apiKey);
$model = 'gpt-4o-mini';

$agent = new Agent($platform, $model);
```

### Running the Agent

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messages = new MessageBag(
    Message::forSystem('You are a helpful chatbot answering questions about LLM agent.'),
    Message::ofUser('Hello, how are you?'),
);
$result = $agent->call($messages);

echo $result->getContent(); // "I'm fine, thank you. How can I help you today?"
```

## Tools

Tools are services that can be called by the LLM to provide additional features or process data.

### Creating Custom Tools

Use the `#[AsTool]` attribute to define a tool:

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

### Enabling Tool Calling

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\Toolbox;

$yourTool = new YourTool();

$toolbox = new Toolbox([$yourTool]);
$toolProcessor = new AgentProcessor($toolbox);

$agent = new Agent(
    $platform,
    $model,
    inputProcessors: [$toolProcessor],
    outputProcessors: [$toolProcessor]
);
```

### Tool Methods

Define multiple tools per class by specifying the method:

```php
use Symfony\AI\Agent\Toolbox\Attribute\AsTool;

#[AsTool(
    name: 'weather_current',
    description: 'get current weather for a location',
    method: 'current',
)]
#[AsTool(
    name: 'weather_forecast',
    description: 'get weather forecast for a location',
    method: 'forecast',
)]
final readonly class OpenMeteo
{
    public function current(float $latitude, float $longitude): array
    {
        // ...
    }

    public function forecast(float $latitude, float $longitude): array
    {
        // ...
    }
}
```

### Tool Parameters & Validation

#### Using the `#[With]` Attribute

```php
use Symfony\AI\Agent\Toolbox\Attribute\AsTool;
use Symfony\AI\Platform\Contract\JsonSchema\Attribute\With;

#[AsTool('my_tool', 'Example tool with parameters requirements.')]
final class MyTool
{
    /**
     * @param string $name   The name of an object
     * @param int    $number The number of an object
     * @param array<string> $categories List of valid categories
     */
    public function __invoke(
        #[With(pattern: '/([a-z0-1]){5}/')]
        string $name,
        #[With(minimum: 0, maximum: 10)]
        int $number,
        #[With(enum: ['tech', 'business', 'science'])]
        array $categories,
    ): string {
        // ...
    }
}
```

#### Automatic Enum Validation

```php
enum Priority: int
{
    case LOW = 1;
    case NORMAL = 5;
    case HIGH = 10;
}

enum ContentType: string
{
    case ARTICLE = 'article';
    case TUTORIAL = 'tutorial';
    case NEWS = 'news';
}

#[AsTool('content_search', 'Search for content with automatic enum validation.')]
final class ContentSearchTool
{
    /**
     * @param array<string> $keywords The search keywords
     * @param ContentType   $type     The content type to search for
     * @param Priority      $priority Minimum priority level
     * @param ContentType|null $fallback Optional fallback content type
     */
    public function __invoke(
        array $keywords,
        ContentType $type,
        Priority $priority,
        ?ContentType $fallback = null,
    ): array {
        // Enums are automatically validated - no #[With] attribute needed!
        // ...
    }
}
```

### Tool Return Values

Tools can return strings, arrays, or objects implementing `JsonSerializable`. Symfony AI automatically converts arrays and objects to JSON strings.

### Third-Party Tools

Register third-party tools using `MemoryToolFactory`:

```php
use Symfony\AI\Agent\Toolbox\Toolbox;
use Symfony\AI\Agent\Toolbox\ToolFactory\MemoryToolFactory;
use Symfony\Component\Clock\Clock;

$metadataFactory = (new MemoryToolFactory())
    ->addTool(Clock::class, 'clock', 'Get the current date and time', 'now');
$toolbox = new Toolbox($metadataFactory, [new Clock()]);
```

### Combining Tool Factories

```php
use Symfony\AI\Agent\Toolbox\Toolbox;
use Symfony\AI\Agent\Toolbox\ToolFactory\ChainFactory;
use Symfony\AI\Agent\Toolbox\ToolFactory\MemoryToolFactory;
use Symfony\AI\Agent\Toolbox\ToolFactory\ReflectionToolFactory;

$reflectionFactory = new ReflectionToolFactory(); // #[AsTool] tools
$metadataFactory = (new MemoryToolFactory())      // Explicit registration
    ->addTool(...);
$toolbox = new Toolbox(new ChainFactory($metadataFactory, $reflectionFactory), [...]);
```

### Subagents

Use one agent as a tool in another:

```php
use Symfony\AI\Agent\Toolbox\Tool\Subagent;
use Symfony\AI\Agent\Toolbox\Toolbox;
use Symfony\AI\Agent\Toolbox\ToolFactory\MemoryToolFactory;

$subagent = new Subagent($agent);
$metadataFactory = (new MemoryToolFactory())
    ->addTool($subagent, 'research_agent', 'Meaningful description for sub-agent');
$toolbox = new Toolbox($metadataFactory, [$subagent]);
```

### Fault Tolerance

Handle errors gracefully with `FaultTolerantToolbox`:

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\FaultTolerantToolbox;

$toolbox = new FaultTolerantToolbox($innerToolbox);
$toolProcessor = new AgentProcessor($toolbox);

$agent = new Agent($platform, $model, inputProcessor: [$toolProcessor], outputProcessor: [$toolProcessor]);
```

#### Custom Exception Handling

```php
use Symfony\AI\Agent\Toolbox\Exception\ToolExecutionExceptionInterface;

class EntityNotFoundException extends \RuntimeException implements ToolExecutionExceptionInterface
{
    public function __construct(
        private string $entityName,
        private int $id,
    ) {}

    public function getToolCallResult(): string
    {
        return sprintf('No %s found with id %d', $this->entityName, $this->id);
    }
}

#[AsTool('get_user_age', 'Get age by user id')]
class GetUserAge
{
    public function __construct(
        private UserRepository $userRepository,
    ) {}

    public function __invoke(int $id): int
    {
        $user = $this->userRepository->find($id);

        if (null === $user) {
            throw new EntityNotFoundException('user', $id);
        }

        return $user->getAge();
    }
}
```

### Tool Sources

Expose data sources from tools:

```php
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\Toolbox;

$toolbox = new Toolbox([new MyTool()]);
$toolProcessor = new AgentProcessor($toolbox, includeSources: true);
```

Implement `HasSourcesInterface` in your tool:

```php
use Symfony\AI\Agent\Toolbox\Source\HasSourcesInterface;
use Symfony\AI\Agent\Toolbox\Source\HasSourcesTrait;

#[AsTool('my_tool', 'Example tool with sources.')]
final class MyTool implements HasSourcesInterface
{
    use HasSourcesTrait;

    public function __invoke(string $query): string
    {
        $this->addSource(
            new Source('Example Source 1', 'https://example.com/source1', 'Relevant content from source 1'),
        );

        // return result
    }
}
```

Retrieve sources from results:

```php
$result = $agent->call($messages);

foreach ($result->getMetadata()->get('sources', []) as $source) {
    echo sprintf(' - %s (%s): %s', $source->getName(), $source->getReference(), $source->getContent());
}
```

### Tool Filtering

Limit tools for specific agent calls:

```php
$this->agent->call($messages, ['tools' => ['tavily_search']]);
```

### Tool Result Interception

React to tool results using event listeners:

```php
$eventDispatcher->addListener(ToolCallsExecuted::class, function (ToolCallsExecuted $event): void {
    foreach ($event->toolCallResults as $toolCallResult) {
        if (str_starts_with($toolCallResult->toolCall->name, 'weather_')) {
            $event->result = new ObjectResult($toolCallResult->result);
        }
    }
});
```

### Tool Call Lifecycle Events

```php
$eventDispatcher->addListener(ToolCallArgumentsResolved::class, function (ToolCallArgumentsResolved $event): void {
    // Tool arguments resolved
});

$eventDispatcher->addListener(ToolCallSucceeded::class, function (ToolCallSucceeded $event): void {
    // Tool execution succeeded with result
});

$eventDispatcher->addListener(ToolCallFailed::class, function (ToolCallFailed $event): void {
    // Tool execution failed with exception
});
```

### Keeping Tool Messages

Preserve tool messages in conversation context:

```php
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\Toolbox;

$toolbox = new Toolbox([$tool]);
$toolProcessor = new AgentProcessor($toolbox, keepToolMessages: true);

$agent = new Agent($platform, $model, inputProcessor: [$toolProcessor], outputProcessor: [$toolProcessor]);
$result = $agent->call($messages);
// $messages now includes tool messages
```

## Retrieval Augmented Generation (RAG)

Use the `SimilaritySearch` tool with the Store component:

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Agent\Bridge\SimilaritySearch\SimilaritySearch;
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\Toolbox;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$similaritySearch = new SimilaritySearch($model, $store);
$toolbox = new Toolbox([$similaritySearch]);
$processor = new AgentProcessor($toolbox);
$agent = new Agent($platform, $model, [$processor], [$processor]);

$messages = new MessageBag(
    Message::forSystem(<<<PROMPT
        Please answer all user questions only using the similarity_search tool.
        Do not add information and if you cannot find an answer, say so.
        PROMPT),
    Message::ofUser('...') // The user's question.
);
$result = $agent->call($messages);
```

## Input & Output Processing

### InputProcessor

Modify messages and options before LLM execution:

```php
use Symfony\AI\Agent\Input;
use Symfony\AI\Agent\InputProcessorInterface;
use Symfony\AI\Platform\Message\AssistantMessage;

final class MyProcessor implements InputProcessorInterface
{
    public function processInput(Input $input): void
    {
        // Mutate options
        $options = $input->getOptions();
        $options['foo'] = 'bar';
        $input->setOptions($options);

        // Mutate MessageBag
        $input->messages->append(new AssistantMessage(sprintf('Please answer using the locale %s', $this->locale)));
    }
}
```

### OutputProcessor

Modify results after LLM execution:

```php
use Symfony\AI\Agent\Output;
use Symfony\AI\Agent\OutputProcessorInterface;

final class MyProcessor implements OutputProcessorInterface
{
    public function processOutput(Output $output): void
    {
        // Mutate result
        if (str_contains($output->result->getContent(), self::STOP_WORD)) {
            $output->result = new TextResult('Sorry, we were unable to find relevant information.');
        }
    }
}
```

### Agent Awareness

Access the agent instance from processors:

```php
use Symfony\AI\Agent\AgentAwareInterface;
use Symfony\AI\Agent\AgentAwareTrait;
use Symfony\AI\Agent\Output;
use Symfony\AI\Agent\OutputProcessorInterface;

final class MyProcessor implements OutputProcessorInterface, AgentAwareInterface
{
    use AgentAwareTrait;

    public function processOutput(Output $out): void
    {
        // Additional agent interaction
        $result = $this->agent->call(...);
    }
}
```

## Agent Memory Management

### Using Memory

Add contextual memory to agent conversations:

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Agent\Memory\MemoryInputProcessor;
use Symfony\AI\Agent\Memory\StaticMemoryProvider;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$personalFacts = new StaticMemoryProvider(
    'My name is Wilhelm Tell',
    'I wish to be a swiss national hero',
    'I am struggling with hitting apples but want to be professional with the bow and arrow',
);
$memoryProcessor = new MemoryInputProcessor($personalFacts);

$agent = new Agent($platform, $model, [$memoryProcessor]);
$messages = new MessageBag(Message::ofUser('What do we do today?'));
$result = $agent->call($messages);
```

### Static Memory

Fixed information available to the agent:

```php
use Symfony\AI\Agent\Memory\StaticMemoryProvider;

$staticMemory = new StaticMemoryProvider(
    'The user is allergic to nuts',
    'The user prefers brief explanations',
);
```

### Embedding Provider

Vector-based memory for retrieving relevant knowledge:

```php
use Symfony\AI\Agent\Memory\EmbeddingProvider;

$embeddingsMemory = new EmbeddingProvider(
    $platform,
    $embeddings, // Your embeddings model
    $store       // Your vector store
);
```

### Dynamic Memory Control

Disable memory for specific calls:

```php
$result = $agent->call($messages, [
    'use_memory' => false, // Disable memory for this call
]);
```

## Testing

### MockAgent

Test agents without making API calls:

```php
use Symfony\AI\Agent\MockAgent;
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$agent = new MockAgent([
    'What is Symfony?' => 'Symfony is a PHP web framework',
    'Tell me about caching' => 'Symfony provides powerful caching',
]);

$messages = new MessageBag(Message::ofUser('What is Symfony?'));
$result = $agent->call($messages);

echo $result->getContent(); // "Symfony is a PHP web framework"
```

### Call Tracking and Assertions

```php
$agent->assertCallCount(1);
$agent->assertCalledWith('What is Symfony?');

$calls = $agent->getCalls();
$lastCall = $agent->getLastCall();

$agent->reset();
```

### MockResponse Objects

```php
use Symfony\AI\Agent\MockResponse;

$complexResponse = new MockResponse('Detailed response content');
$agent = new MockAgent([
    'complex query' => $complexResponse,
    'simple query' => 'Simple string response',
]);
```

### Callable Responses

```php
$agent = new MockAgent();

$agent->addResponse('weather', function ($messages, $options, $input) {
    $messageCount = count($messages->getMessages());
    return "Weather info (context: {$messageCount} messages)";
});

$agent->addResponse('complex', function ($messages, $options, $input) {
    return new MockResponse("Complex response for: {$input}");
});
```

### Service Testing Example

```php
class ChatServiceTest extends TestCase
{
    public function testChatResponse(): void
    {
        $agent = new MockAgent([
            'Hello' => 'Hi there! How can I help?',
        ]);

        $chatService = new ChatService($agent);
        $response = $chatService->processMessage('Hello');

        $this->assertSame('Hi there! How can I help?', $response);
        $agent->assertCallCount(1);
        $agent->assertCalledWith('Hello');
    }
}
```

## Built-in Tools

The component includes example tools:
- Brave Tool
- Clock Tool
- Crawler Tool
- Mapbox Geocode Tool
- Mapbox Reverse Geocode Tool
- SerpAPI Tool
- Tavily Tool
- Weather Tool with Event Listener
- Wikipedia Tool
- YouTube Transcriber Tool

---

**License**: This work is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.
