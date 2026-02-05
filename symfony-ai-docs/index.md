# Symfony AI Documentation

Welcome to the Symfony AI documentation. Symfony AI provides a comprehensive set of components for building AI-powered applications with PHP.

## Components

- [Platform](components/platform.md) - Unified abstraction for interacting with different AI models and providers
- [Agent](components/agent.md) - Framework for building AI agents with tools and workflows
- [Chat](components/chat.md) - API for storing and retrieving chat messages
- [Store](components/store.md) - Low-level abstraction for vector stores and RAG
- [Mate](components/mate.md) - Development tool for creating local MCP servers

## Bundles

- [AI Bundle](bundles/ai-bundle.md) - Symfony's official integration bundle for AI components
- [MCP Bundle](bundles/mcp-bundle.md) - Symfony integration for Model Context Protocol

## Cookbook

- [Building a Chatbot with Memory](cookbook/chatbot-with-memory.md) - Build chatbots that remember user preferences
- [Implementing RAG](cookbook/rag-implementation.md) - Complete RAG systems with vector stores

## Installation

```bash
composer require symfony/ai-bundle
```

## Quick Start

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
            Message::forSystem('You are a helpful assistant.'),
            Message::ofUser($message),
        );

        return $this->agent->call($messages)->getContent();
    }
}
```

## License

This documentation is licensed under [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
