# Discovery

## Overview

Tempest automatically locates controller actions, event handlers, console commands, and other application components without requiring manual configuration. This automatic scanning process is called **discovery** and is powered by composer metadata.

The framework analyzes file names, attributes, interfaces, and return types to determine component purpose. For example, web routes are discovered through route attributes on methods.

## Discovery in Production

Discovery is cached in production to avoid scanning files on every request. Before deployment, run:

```bash
./tempest discovery:generate --no-interaction
```

This ensures the discovery cache is up-to-date and ready for production use.

## Discovery for Local Development

During development, discovery only operates on application code. The cache should regenerate when packages are installed or updated by adding this to `composer.json`:

```json
{
  "scripts": {
    "post-package-update": [
      "@php tempest discovery:generate"
    ]
  }
}
```

### Disabling Discovery Cache

Set the environment variable to enable discovery for vendor code during development:

```bash
DISCOVERY_CACHE=false
```

This is useful when developing third-party packages alongside your application.

### Troubleshooting

Use `discovery:clear` to reset the cache, which rebuilds on the next framework boot. For corrupted caches, manually delete `.tempest/cache/discovery`.

## Implementing Custom Discovery

Custom discovery classes implement the `Discovery` interface with `discover()` and `apply()` methods. The `IsDiscovery` trait provides default implementations.

### Discovering Classes

The `discover()` method receives `DiscoveryLocation` and `ClassReflector` parameters. Use the reflector to analyze class attributes, methods, and parameters, then register matches using `$this->discoveryItems->add()`.

Example event bus discovery pattern:

```php
use Tempest\Discovery\Discovery;
use Tempest\Discovery\IsDiscovery;

final class EventBusDiscovery implements Discovery
{
    use IsDiscovery;

    public function discover(DiscoveryLocation $location, ClassReflector $class): void
    {
        foreach ($class->getPublicMethods() as $method) {
            $eventHandler = $method->getAttribute(EventHandler::class);
            // Validation logic...
            $this->discoveryItems->add($location, [$eventName, $eventHandler, $method]);
        }
    }

    public function apply(): void
    {
        foreach ($this->discoveryItems as [$eventName, $eventHandler, $method]) {
            $this->eventBusConfig->addClassMethodHandler(
                event: $eventName,
                handler: $eventHandler,
                reflectionMethod: $method,
            );
        }
    }
}
```

### Discovering Files

Implement `DiscoversPath` interface for non-PHP files like views or migrations:

```php
use Tempest\Discovery\Discovery;
use Tempest\Discovery\DiscoversPath;
use Tempest\Discovery\IsDiscovery;

final class ViteDiscovery implements Discovery, DiscoversPath
{
    use IsDiscovery;

    public function discover(DiscoveryLocation $location, ClassReflector $class): void
    {
        return;
    }

    public function discoverPath(DiscoveryLocation $location, string $path): void
    {
        if (!Str\ends_with($path, ['.ts', '.css', '.js'])) {
            return;
        }
        if (!str($path)->beforeLast('.')->endsWith('.entrypoint')) {
            return;
        }
        $this->discoveryItems->add($location, [$path]);
    }

    public function apply(): void
    {
        foreach ($this->discoveryItems as [$path]) {
            $this->viteConfig->addEntrypoint($path);
        }
    }
}
```

## Excluding Files and Classes

Use `DiscoveryConfig` to exclude specific items:

```php
use Tempest\Core\DiscoveryConfig;

return new DiscoveryConfig()
    ->skipClasses(GlobalHiddenDiscovery::class)
    ->skipPaths(__DIR__ . '/../../Fixtures/GlobalHiddenPathDiscovery.php');
```

## Built-in Discovery Classes

Tempest includes discovery classes for:

- **RouteDiscovery** — controller actions
- **ConsoleCommandDiscovery** — console commands
- **EventBusDiscovery** — event handlers
- **CommandBusDiscovery** — command handlers
- **MigrationDiscovery** — database migrations
- **ViewComponentDiscovery** — view components
- **ViteDiscovery** — asset entrypoints
- **MapperDiscovery** — data mappers
- **CasterDiscovery** & **SerializerDiscovery** — data transformation
- **PolicyDiscovery** — access control policies
- **InitializerDiscovery** — dependency initializers
- **ScheduleDiscovery** — scheduled tasks
- **InsightsProviderDiscovery** — framework insights
