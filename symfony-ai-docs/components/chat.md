# Symfony AI - Chat Component Documentation

## Overview
The Chat component provides an API to interact with agents, allowing you to store messages and retrieve them later for future chat and context-retrieving purposes.

## Installation

```bash
$ composer require symfony/ai-chat
```

## Basic Usage

To initiate a chat, instantiate the `Symfony\AI\Chat\Chat` class with a `Symfony\AI\Agent\AgentInterface` and a `Symfony\AI\Chat\MessageStoreInterface`:

```php
use Symfony\AI\Agent\Agent;
use Symfony\AI\Chat\Chat;
use Symfony\AI\Chat\InMemory\Store as InMemoryStore;
use Symfony\AI\Platform\Bridge\OpenAi\Gpt;
use Symfony\AI\Platform\Bridge\OpenAi\PlatformFactory;
use Symfony\AI\Platform\Message\Message;

$platform = PlatformFactory::create($apiKey);

$agent = new Agent($platform, 'gpt-4o-mini');
$chat = new Chat($agent, new InMemoryStore());

$chat->submit(Message::ofUser('Hello'));
```

## Supported Message Stores

The Chat component supports multiple message store backends:

- **Cache** - Symfony Cache component
- **Cloudflare** - Cloudflare KV storage
- **Doctrine DBAL** - Database abstraction layer
- **HttpFoundation session** - Session-based storage
- **InMemory** - PHP arrays (in-process only)
- **Meilisearch** - Search engine storage
- **MongoDb** - NoSQL database
- **Pogocache** - Distributed caching
- **Redis** - Key-value store
- **SurrealDb** - Cloud database

## Implementing a Custom Bridge

Create a custom message store by implementing the `MessageStoreInterface`:

```php
use Symfony\AI\Chat\MessageStoreInterface;
use Symfony\AI\Platform\Message\MessageBag;

class MyCustomStore implements MessageStoreInterface
{
    public function save(MessageBag $messages): void
    {
        // Implementation to add a message bag to the store
    }

    public function load(): MessageBag
    {
        // Implementation to return a message bag from the store
    }
}
```

## Managing a Store

For stores requiring initialization (tables, indexes, etc.), implement `ManagedStoreInterface`:

```php
use Symfony\AI\Chat\ManagedStoreInterface;
use Symfony\AI\Chat\MessageStoreInterface;

class MyCustomStore implements ManagedStoreInterface, MessageStoreInterface
{
    public function setup(array $options = []): void
    {
        // Implementation to create the store
    }

    public function drop(): void
    {
        // Implementation to drop the store (and related messages)
    }
}
```

## Console Commands

When using the Chat component in a Symfony application with AiBundle, use these commands:

```bash
# Setup the message store
$ php bin/console ai:message-store:setup <store-name>

# Drop the message store
$ php bin/console ai:message-store:drop <store-name>
```

### Configuration Example

```yaml
# config/packages/ai.yaml
ai:
    message_store:
        cache:
            symfonycon:
                service: 'cache.app'
```

Then use:
```bash
$ php bin/console ai:message-store:setup symfonycon
$ php bin/console ai:message-store:drop symfonycon
```

## Advanced Examples

The repository includes comprehensive examples for different storage backends:
- Persistent chat with Cache
- Long-term context with Doctrine DBAL, MongoDB, Redis, SurrealDb, Meilisearch, Cloudflare
- Session-based storage with HttpFoundation
- In-memory storage for single process context
- Pogocache integration

---

**License**: This work is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.
