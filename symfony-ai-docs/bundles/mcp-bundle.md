# MCP Bundle Documentation

## Overview

The **MCP Bundle** is a Symfony integration bundle for the [Model Context Protocol](https://modelcontextprotocol.io/) using the official MCP SDK [mcp/sdk](https://github.com/modelcontextprotocol/php-sdk).

**Capabilities:**
- Tools, prompts, and resources support
- Server functionality via HTTP transport and STDIO
- Resource templates (awaiting MCP SDK support)

---

## Installation

```bash
$ composer require symfony/mcp-bundle
```

---

## Usage

### Configuration Setup

Add routing configuration:

```yaml
# config/routes.yaml
mcp:
    resource: .
    type: mcp
```

---

## Act as Server

Expose tools, prompts, resources, and resource templates to clients like Claude Desktop by configuring the `client_transports` section with STDIO or HTTP.

### Creating MCP Capabilities

MCP capabilities are automatically discovered using PHP attributes.

#### Tools

Actions that can be executed:

```php
use Mcp\Capability\Attribute\McpTool;

class CurrentTimeTool
{
    #[McpTool(name: 'current-time')]
    public function getCurrentTime(string $format = 'Y-m-d H:i:s'): string
    {
        return (new \DateTime('now', new \DateTimeZone('UTC')))->format($format);
    }
}
```

#### Prompts

System instructions for AI context:

```php
use Mcp\Capability\Attribute\McpPrompt;

class TimePrompts
{
    #[McpPrompt(name: 'time-analysis')]
    public function getTimeAnalysisPrompt(): array
    {
        return [
            ['role' => 'user', 'content' => 'You are a time management expert.']
        ];
    }
}
```

#### Resources

Static data that can be read:

```php
use Mcp\Capability\Attribute\McpResource;

class TimeResource
{
    #[McpResource(uri: 'time://current', name: 'current-time')]
    public function getCurrentTimeResource(): array
    {
        return [
            'uri' => 'time://current',
            'mimeType' => 'text/plain',
            'text' => (new \DateTime('now'))->format('Y-m-d H:i:s')
        ];
    }
}
```

#### Resource Templates

Dynamic resources with parameters:

```php
use Mcp\Capability\Attribute\McpResourceTemplate;

class TimeResourceTemplate
{
    #[McpResourceTemplate(uriTemplate: 'time://{timezone}', name: 'time-by-timezone')]
    public function getTimeByTimezone(string $timezone): array
    {
        $time = (new \DateTime('now', new \DateTimeZone($timezone)))->format('Y-m-d H:i:s T');
        return [
            'uri' => "time://$timezone",
            'mimeType' => 'text/plain',
            'text' => $time
        ];
    }
}
```

> **Note:** Resource Templates are not yet functional as the MCP SDK is missing required handlers. See [MCP SDK issue #9](https://github.com/modelcontextprotocol/php-sdk/issues/9).

### Attribute Placement Patterns

**Invokable Pattern** - Attribute on class with `__invoke()`:

```php
#[McpTool(name: 'my-tool')]
class MyTool
{
    public function __invoke(string $param): string
    {
        // Implementation
    }
}
```

**Method-Based Pattern** - Multiple attributes on individual methods:

```php
class MyTools
{
    #[McpTool(name: 'tool-one')]
    public function toolOne(): string { }

    #[McpTool(name: 'tool-two')]
    public function toolTwo(): string { }
}
```

### Transport Types

- **STDIO Transport** - For command-line clients (e.g., `symfony console mcp:server`)
- **HTTP Transport** - For web-based clients and MCP Inspector using streamable HTTP connections

HTTP transport features:
- JSON-RPC 2.0 over HTTP POST requests
- Session management with configurable storage (file/memory)
- CORS headers for cross-origin requests
- Proper MCP initialization handshake

---

## Act as Client

> **Warning:** Not implemented yet, but planned for the future.

To use your application as an MCP client, configure the `servers` you want to connect to using STDIO or HTTP transport.

See [MCP Server List](https://modelcontextprotocol.io/examples) for available servers.

---

## Configuration

```yaml
# config/packages/mcp.yaml
mcp:
    app: 'app' # Application name to be exposed to clients
    version: '1.0.0' # Application version
    description: 'A sample MCP server for time management.'
    icons:
        - src: 'https://example.com/icon.png'
          mime_type: 'image/png'
          sizes: ['64x64']
    website_url: 'https://example.com'
    pagination_limit: 50 # Maximum number of items returned per list request
    instructions: |
        This server provides time management capabilities for developers.

        Use when working with timestamps, time zones, or time-based calculations.
        All timestamps are in UTC unless specified otherwise.

    client_transports:
        stdio: true # Enable STDIO via command
        http: true # Enable HTTP transport via controller

    # HTTP transport configuration (optional)
    http:
        path: /_mcp # HTTP endpoint path (default: /_mcp)
        session:
            store: file # Session store type: 'file' or 'memory'
            directory: '%kernel.cache_dir%/mcp-sessions'
            ttl: 3600 # Session TTL in seconds

    # Not supported yet
    servers:
        name:
            transport: 'stdio' # 'stdio' or 'http'
            stdio:
                command: 'php /path/bin/console mcp:server'
                arguments: []
            http:
                url: 'http://localhost:8000/_mcp'
```

---

## Logging Configuration

Configure MCP-specific logging in `config/packages/monolog.yaml`:

```yaml
# config/packages/monolog.yaml
monolog:
    channels: ['mcp']
    handlers:
        mcp:
            type: rotating_file
            path: '%kernel.logs_dir%/mcp.log'
            level: info
            channels: ['mcp']
            max_files: 30
```

Environment-specific logging example:

```yaml
monolog:
    handlers:
        mcp_dev:
            type: stream
            path: '%kernel.logs_dir%/mcp.log'
            level: debug
            channels: ['mcp']
        mcp_prod:
            type: slack
            level: error
            channels: ['mcp']
            webhook_url: '%env(SLACK_WEBHOOK)%'
```

---

## Profiler

When the Symfony Web Profiler is enabled, the MCP Bundle adds a dedicated panel showing all registered capabilities:

- **Tools** - All registered MCP tools with descriptions and input schemas
- **Prompts** - Available prompts with arguments and requirements
- **Resources** - Static resources with URIs and MIME types
- **Resource Templates** - Dynamic resource templates with URI patterns

---

## Event System

The MCP Bundle automatically configures the Symfony EventDispatcher to work with the MCP SDK's event system.

### Available Events

- `Mcp\Event\ToolListChangedEvent` - When a tool is registered
- `Mcp\Event\ResourceListChangedEvent` - When a resource is registered
- `Mcp\Event\ResourceTemplateListChangedEvent` - When a resource template is registered
- `Mcp\Event\PromptListChangedEvent` - When a prompt is registered

### Listening to Events

```php
use Mcp\Event\ToolListChangedEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener]
class McpCapabilityListener
{
    public function onToolListChanged(ToolListChangedEvent $event): void
    {
        // Handle tool registration
        // For example: invalidate cache, log changes, notify clients
    }
}
```

---

**License**: This work is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.
