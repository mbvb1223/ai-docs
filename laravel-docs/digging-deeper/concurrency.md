# Laravel 12.x Concurrency Documentation

## Introduction

Laravel's `Concurrency` facade provides a simple API for executing closures concurrently in separate PHP processes.

## How it Works

Laravel achieves concurrency by:
1. Serializing the given closures
2. Dispatching them to a hidden Artisan CLI command
3. Invoking the closures within their own PHP processes
4. Serializing the results back to the parent process

## Available Drivers

- **`process`** (default) - Works in all contexts
- **`fork`** - Better performance, CLI-only (requires `spatie/fork`)
- **`sync`** - Executes sequentially (useful for testing)

```bash
composer require spatie/fork
```

## Running Concurrent Tasks

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

### Specifying a Driver

```php
$results = Concurrency::driver('fork')->run(...);
```

### Configuration

```bash
php artisan config:publish concurrency
```

## Deferring Concurrent Tasks

Execute closures after HTTP response is sent:

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/concurrency)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
