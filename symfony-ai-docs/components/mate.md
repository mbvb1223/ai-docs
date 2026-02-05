# Symfony AI - Mate Component Documentation

## Overview

The Mate component is a **development tool** that creates a local MCP (Model Context Protocol) server, enabling AI assistants (Claude, GitHub Copilot, JetBrains AI, Cursor, etc.) to interact with PHP applications through standardized tools. It's **not intended for production use**.

## Installation

```bash
composer require --dev symfony/ai-mate
```

## Quick Start

### Initialize Configuration

```bash
vendor/bin/mate init
```

This creates:
- `mate/` directory with configuration files
- `mate/src` directory for custom extensions
- `mcp.json` for MCP client configuration
- Updates `composer.json` with proper autoload configuration

### Update Autoloader

```bash
composer dump-autoload
```

### Discover Extensions

```bash
vendor/bin/mate discover
```

### Start the MCP Server

```bash
vendor/bin/mate serve
```

## Adding Custom Tools

Create tools in `mate/src` using the `#[McpTool]` attribute:

```php
// mate/MyTool.php
namespace App\Mate;

use Mcp\Capability\Attribute\McpTool;

class MyTool
{
    #[McpTool(name: 'my_tool', description: 'My custom tool')]
    public function execute(string $param): array
    {
        return ['result' => $param];
    }
}
```

## Configuration

### Extensions Configuration (`mate/extensions.php`)

```php
// This file is managed by 'mate discover'
// You can manually edit to enable/disable extensions

return [
    'vendor/package-name' => ['enabled' => true],
    'vendor/another-package' => ['enabled' => false],
];
```

### Services Configuration (`mate/config.php`)

```php
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $container->parameters()
        // Override default parameters here
        // ->set('mate.cache_dir', sys_get_temp_dir().'/mate')
        // ->set('mate.env_file', ['.env'])
    ;

    $container->services()
        // Register your custom services here
    ;
};
```

### Disabling Specific Features

```php
use Symfony\AI\Mate\Container\MateHelper;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    MateHelper::disableFeatures($container, [
        'symfony/ai-mate' => ['php-version', 'operating-system'],
    ]);
};
```

## Available Bridges

### Symfony Bridge (`symfony/ai-symfony-mate-extension`)

#### Container Introspection

**Tool:** `symfony-services` - List all Symfony services from the compiled container

**Configuration:**
```php
$container->parameters()
    ->set('ai_mate_symfony.cache_dir', '%root_dir%/var/cache');
```

#### Profiler Data Access

Available when `symfony/http-kernel` and `symfony/web-profiler-bundle` are installed.

**Tools:**
- `symfony-profiler-list` - List available profiler profiles
- `symfony-profiler-latest` - Get latest profile summary
- `symfony-profiler-search` - Search profiles by criteria
- `symfony-profiler-get` - Get specific profile by token

**Resources:**
- `symfony-profiler://profile/{token}` - Full profile details
- `symfony-profiler://profile/{token}/{collector}` - Collector-specific data

**Configuration (single directory):**
```php
$container->parameters()
    ->set('ai_mate_symfony.profiler_dir', '%mate.root_dir%/var/cache/dev/profiler');
```

**Configuration (multiple directories):**
```php
$container->parameters()
    ->set('ai_mate_symfony.profiler_dir', [
        'website' => '%mate.root_dir%/var/cache/website/dev/profiler',
        'admin' => '%mate.root_dir%/var/cache/admin/dev/profiler',
    ]);
```

**Example: Search for errors**
```json
{
    "method": "tools/call",
    "params": {
        "name": "symfony-profiler-search",
        "arguments": {
            "statusCode": 500,
            "limit": 20
        }
    }
}
```

### Monolog Bridge (`symfony/ai-monolog-mate-extension`)

**Tools:**
- `monolog-search` - Search log entries by text term
- `monolog-search-regex` - Search using regex patterns
- `monolog-context-search` - Search by context field value
- `monolog-tail` - Get last N log entries
- `monolog-list-files` - List available log files
- `monolog-list-channels` - List all log channels
- `monolog-by-level` - Get entries filtered by level

**Configuration:**
```php
$container->parameters()
    ->set('ai_mate_monolog.log_dir', '%root_dir%/var/log');
```

## Built-in Tools

Core package provides:
- `php-version` - Get PHP version
- `operating-system` - Get OS
- `operating-system-family` - Get OS family
- `php-extensions` - List loaded PHP extensions

## Commands

### `mate init`
Initialize AI Mate configuration and create `mate/` directory.

### `mate discover`
Scan for MCP extensions and generate/update `mate/extensions.php`.

### `mate serve`
Start the MCP server with stdio transport.

### `mate clear-cache`
Clear the MCP server cache.

### `mate debug:capabilities`
Display all discovered MCP capabilities grouped by extension.

**Options:**
```bash
--format=FORMAT          # text (default) or json
--extension=EXTENSION    # Filter by package name
--type=TYPE             # tool, resource, prompt, or template
```

**Examples:**
```bash
vendor/bin/mate debug:capabilities
vendor/bin/mate debug:capabilities --type=tool
vendor/bin/mate debug:capabilities --extension=symfony/ai-monolog-mate-extension
vendor/bin/mate debug:capabilities --format=json
```

### `mate debug:extensions`
Display detailed information about discovered and loaded extensions.

**Options:**
```bash
--format=FORMAT   # text (default) or json
--show-all        # Show disabled extensions
```

### `mate mcp:tools:list`
List all available MCP tools with metadata.

**Options:**
```bash
--filter=PATTERN       # Filter by name pattern
--extension=EXTENSION  # Filter by extension
--format=FORMAT        # table (default) or json
```

**Examples:**
```bash
vendor/bin/mate mcp:tools:list
vendor/bin/mate mcp:tools:list --filter="monolog*"
vendor/bin/mate mcp:tools:list --extension=symfony/ai-monolog-mate-extension
```

### `mate mcp:tools:inspect`
Display detailed information about a specific tool.

```bash
vendor/bin/mate mcp:tools:inspect php-version
vendor/bin/mate mcp:tools:inspect php-version --format=json
```

### `mate mcp:tools:call`
Execute MCP tools via JSON input parameters.

**Options:**
```bash
--format=FORMAT  # pretty (default) or json
```

**Examples:**
```bash
vendor/bin/mate mcp:tools:call php-version '{}'
vendor/bin/mate mcp:tools:call search-logs '{"query": "error", "level": "error"}'
```

## Security

- **No vendor extensions are enabled by default** - explicitly enable packages in `mate/extensions.php`
- Local `mate/` directory is always enabled for rapid development
- Profiler data automatically redacts: cookies, session data, authentication headers, and sensitive environment variables

## Adding Third-Party Extensions

1. Install the package:
   ```bash
   composer require vendor/symfony-tools
   ```

2. Discover available tools:
   ```bash
   vendor/bin/mate discover
   ```

3. Enable/disable in `mate/extensions.php`:
   ```php
   return [
       'vendor/symfony-tools' => ['enabled' => true],
       'vendor/unwanted-tools' => ['enabled' => false],
   ];
   ```

## Troubleshooting

### Container Not Found
- Ensure cache directory parameter points to correct location
- Look for compiled container XML file (e.g., `App_KernelDevDebugContainer.xml`)

### Services Not Appearing
1. Clear Symfony cache: `bin/console cache:clear`
2. Ensure container is compiled (warm up cache)
3. Verify container XML file exists

### Profiles Not Found
1. Verify profiler directory parameter is correct
2. Ensure Symfony profiler is enabled
3. Generate HTTP requests to create profile data

### Logs Not Found
- Ensure log directory parameter points to correct location where Monolog files are stored

---

**License**: This work is licensed under a [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) license.
