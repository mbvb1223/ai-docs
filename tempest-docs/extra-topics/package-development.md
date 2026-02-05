# Package Development

## Overview

Developing packages for Tempest requires adding `tempest/core` as a dependency. The framework's discovery system automatically locates and registers discoverable classes through composer metadata. Unlike competing frameworks, Tempest eschews dedicated service providers in favor of leveraging its discovery and initializer mechanisms.

## Preventing Discovery

Developers can exclude classes from automatic discovery by applying the `Tempest\Discovery\SkipDiscovery` attribute. This allows for creating classes that remain internal or are only exposed when users explicitly publish them.

```php
use Tempest\Discovery\SkipDiscovery;

#[SkipDiscovery]
final readonly class UserMigration implements Migration
{
    // …
}
```

## Installers

Installers publish files to user projects, making them suitable for exporting migrations or other resources that shouldn't be automatically discovered.

Implement `Tempest\Core\Installer` and typically utilize the `Tempest\Core\PublishesFiles` trait for convenient file publishing with automatic import adjustment.

### Publishing Files

The `publish()` method copies files while automatically correcting their namespace references. Users receive prompts to specify destinations and confirm overwrites.

```php
use Tempest\Core\Installer;
use Tempest\Core\PublishesFiles;
use Tempest\Discovery\SkipDiscovery;

final readonly class AuthInstaller implements Installer
{
    use PublishesFiles;

    private(set) string $name = 'auth';

    public function install(): void
    {
        $publishFiles = [
            __DIR__ . '/User.php' => src_path('User.php'),
            __DIR__ . '/UserMigration.php' => src_path('UserMigration.php'),
        ];

        foreach ($publishFiles as $source => $destination) {
            $this->publish(
                source: $source,
                destination: $destination,
            );
        }

        $this->publishImports();
    }
}
```

### Customizing the Publishing Process

Pass a callback to `publish()` for post-copy customization before import adjustment:

```php
$this->publish(
    source: $source,
    destination: $destination,
    callback: function (string $source, string $destination): void {
        // Custom logic here
    },
);

$this->publishImports();
```

### Ensuring Correct Imports

The `publishImports()` method loops through published files and updates any imports referencing them. Call this after all `publish()` operations.

## Provider Classes

While Tempest lacks formal service providers, you can listen to `KernelEvent::BOOTED` for package initialization. This event fires after the kernel boots but before application code runs.

```php
use Tempest\Core\KernelEvent;
use Tempest\EventBus\EventHandler;

final readonly class MyPackageProvider
{
    public function __construct(
        private Container $container,
    ) {}

    #[EventHandler(KernelEvent::BOOTED)]
    public function initialize(): void
    {
        // Package setup logic
    }
}
```

## Testing Helpers

Extend `Tempest\Framework\Testing\IntegrationTest` in PHPUnit tests to automatically boot the framework and access helper methods for testing your package.
