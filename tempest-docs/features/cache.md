# Cache

## Overview

The cache component in Tempest is built on Symfony's Cache, offering access to many different adapters through a convenient, simple interface. By default, filesystem-based caching is used, though alternative backends can be configured.

## Getting Started

Access caching through the `Tempest\Cache\Cache` interface via dependency injection:

```php
use Tempest\Cache\Cache;
use Tempest\DateTime\Duration;

final readonly class OrderService
{
    public function __construct(
        private Cache $cache,
    ) {}

    public function getOrdersCount(): int
    {
        return $this->cache->resolve(
            key: 'orders_count',
            callback: fn () => $this->fetchOrdersCountFromDatabase(),
            expiration: Duration::hours(12)
        );
    }
}
```

## Core Methods

**Getting values:**
```php
$cache->get($key);
```

**Setting values:**
```php
$cache->put($key, $value);
```

**Resolving with callback:**
```php
$cache->resolve($key, function () {
    return $this->expensiveOperation();
});
```

## Cache Management

Clear cache programmatically using `clear()` or via CLI:
```bash
./tempest cache:clear
```

## Configuration & Control

**Disable project cache:**
```
CACHE_ENABLED=false
CACHE_CUSTOM_ENABLED=false
```

**Disable internal caches:**
```
INTERNAL_CACHES=false
VIEW_CACHE=false
ICON_CACHE=false
DISCOVERY_CACHE=false
CONFIG_CACHE=false
```

## Locks

Create and manage locks for concurrent operations:

```php
$lock = $cache->lock('processing', Duration::seconds(30));

if ($lock->acquire()) {
    $this->process();
    $lock->release();
}

// Or with callback:
$lock->execute($this->process(...), wait: Duration::seconds(30));
```

**Lock ownership:**
```php
$cache->lock("processing:{$processId}", owner: $processId)->release();
```

## Testing

Fake cache implementation for tests:

```php
$cache = $this->cache->fake();
$cache->assertCached('users_count');
$cache->assertEmpty();
$cache->assertNotLocked('processing');
```

## Supported Adapters

- FilesystemCacheConfig
- InMemoryCacheConfig
- PhpCacheConfig
