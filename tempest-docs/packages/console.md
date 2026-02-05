# Console Package

## Overview

The Tempest console component functions as a standalone package designed for constructing console applications.

## Installation and Usage

To incorporate the console component independently, install the package via Composer:

```bash
composer require tempest/console
```

After installation, initialize your console application with this setup:

```php
#!/usr/bin/env php
<?php

use Tempest\Console\ConsoleApplication;

require_once __DIR__ . '/vendor/autoload.php';

ConsoleApplication::boot()->run();
```

## Registering Commands

The console component leverages the discovery system to automatically identify and register commands. No manual registration is required—any method decorated with the `#[ConsoleCommand]` attribute becomes automatically discoverable.

Detailed command-building guidance is available in the [console commands documentation](../essentials/console-commands.md).

## Configuring Discovery

The system automatically discovers console commands from three sources:

1. **Core Tempest packages** — Built-in Tempest commands
2. **Vendor packages** — Third-party packages requiring `tempest/framework` or `tempest/core`
3. **App namespaces** — PSR-4 autoload paths configured in `composer.json`

For granular control over discovery locations, pass additional paths to the boot method:

```php
use Tempest\Console\ConsoleApplication;
use Tempest\Discovery\DiscoveryLocation;

ConsoleApplication::boot(
    discoveryLocations: [
        new DiscoveryLocation(
            namespace: 'MyApp\\',
            path: __DIR__ . '/src',
        ),
    ],
)->run();
```

### Boot Method Parameters

- `$name` — Application name (default: `'Tempest'`)
- `$root` — Root directory (default: current working directory)
- `$discoveryLocations` — Additional discovery paths to append
