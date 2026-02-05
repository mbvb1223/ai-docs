# Filament Plugins - Getting Started

## Overview

Filament provides a plugin system to extend functionality either for individual applications or as redistributable packages. The documentation distinguishes between two main plugin contexts:

### Plugin Types

**Panel Plugins** integrate with Panel Builders to add features like:
- Dashboard widgets
- Resource sets (e.g., Blog or User Management features)

**Standalone Plugins** work outside Panel Builders and extend:
- Form builders with custom fields
- Table builders with custom columns or filters

These contexts can coexist within a single plugin.

## Prerequisites

Before building plugins, familiarize yourself with:
1. Laravel Package Development
2. Spatie Package Tools
3. Filament Asset Management

## Key Concepts

### The Plugin Object

Filament uses a Plugin object implementing the `Filament\Contracts\Plugin` interface. This serves as the main configuration entry point for panel-based plugins. However, plugin objects are optional—you can build plugins without them.

**Important note:** "The Plugin object is only used for Panel Providers. Standalone Plugins do not use this object."

### Asset Registration

All asset registration (CSS, JavaScript, Alpine Components) occurs through the plugin's service provider in the `packageBooted()` method, enabling Filament's Asset Manager to load resources when needed.

## Plugin Development Setup

### Quick Start

Use the [Filament Plugin Skeleton](https://github.com/filamentphp/plugin-skeleton) to accelerate development:

1. Click "Use this template" on the GitHub repository
2. Clone the generated repository
3. Run `php ./configure.php` in the project root
4. Answer configuration prompts to scaffold your plugin

## Upgrading Existing Plugins

Key change: Replace `PluginServiceProvider` with `PackageServiceProvider` and add a static `$name` property:

```php
class MyPluginServiceProvider extends PackageServiceProvider
{
    public static string $name = 'my-plugin';

    public function configurePackage(Package $package): void
    {
        $package->name(static::$name);
    }
}
```
