# Standalone Components

## Overview

Multiple Tempest components can be deployed independently in existing or new projects, including `tempest/console`, `tempest/http`, `tempest/event-bus`, `tempest/debug`, and `tempest/command-bus`. However, some components still depend on `tempest/core`, though this may change as the framework matures.

## tempest/console

Installation:
```bash
composer require tempest/console
```

The package includes a built-in binary:
```bash
./vendor/bin/tempest
```

Alternatively, manually initialize the console application:
```php
<?php
use \Tempest\Console\ConsoleApplication;
require_once __DIR__ . '/vendor/autoload.php';
ConsoleApplication::boot()->run();
```

## tempest/http

Installation:
```bash
composer require tempest/http
```

This component handles web application functionality: routing, view rendering, controllers, HTTP exception handling, and view components. The `tempest/console` package comes bundled for managing discovery cache, static pages, and local development.

Set up via console:
```bash
./vendor/bin/tempest install framework
```

Or manually create `public/index.php`:
```php
<?php
use \Tempest\Router\HttpApplication;
require_once __DIR__ . '/vendor/autoload.php';
HttpApplication::boot(
    root: __DIR__ . '/../',
)->run();
```

## tempest/container

Installation:
```bash
composer require tempest/container
```

This standalone container implementation requires manual initializer configuration:
```php
$container = new Tempest\Container\GenericContainer();
$container->addInitializer(FooInitializer::class);
$foo = $container->get(Foo::class);
```

## tempest/debug

Installation:
```bash
composer require tempest/debug
```

Provides debugging functions (`lw`, `ld`, `ll`). Integrates with Tempest logging when installed in a full framework project.

```php
ld($variable);
```

## tempest/view

Tempest View functions as a standalone rendering engine with separate documentation available.

## tempest/event-bus

Installation:
```bash
composer require tempest/event-bus
```

Boot the kernel for handler discovery:
```php
$container = Tempest::boot();
$eventBus = $container->get(\Tempest\EventBus\EventBus::class);
$eventBus->dispatch(new MyEvent());
// Or use helper function:
\Tempest\event(new MyEvent());
```

## tempest/command-bus

Installation:
```bash
composer require tempest/command-bus
```

Similar to the event bus, requires kernel initialization:
```php
$container = Tempest::boot();
$commandBus = $container->get(\Tempest\CommandBus\CommandBus::class);
$commandBus->dispatch(new MyCommand());
// Or use helper function:
\Tempest\command(new MyCommand());
```

## tempest/mapper

Installation:
```bash
composer require tempest/mapper
```

Maps data between various formats (arrays, objects, JSON):
```php
Tempest::boot();
$foo = map(['name' => 'Hi'])->to(Foo::class);
```
