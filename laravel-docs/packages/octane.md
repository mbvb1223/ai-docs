# Laravel 12.x Octane Documentation

## Introduction

Laravel Octane supercharges your application using high-performance servers: FrankenPHP, Swoole, Open Swoole, and RoadRunner.

## Installation

```bash
composer require laravel/octane
php artisan octane:install
```

## Starting the Server

```bash
php artisan octane:start
```

### With File Watching

```bash
npm install --save-dev chokidar
php artisan octane:start --watch
```

### Specify Workers

```bash
php artisan octane:start --workers=4
php artisan octane:start --workers=4 --task-workers=6  # Swoole
```

## Server Management

```bash
php artisan octane:reload    # Reload workers
php artisan octane:status    # Check status
php artisan octane:stop      # Stop server
```

## Nginx Configuration

```nginx
location @octane {
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_pass http://127.0.0.1:8000$suffix;
}
```

## Dependency Injection Best Practices

### Avoid Container Injection

```php
// Don't do this
$this->app->singleton(Service::class, fn ($app) => new Service($app));

// Do this instead
$this->app->singleton(Service::class, fn () => new Service(fn () => Container::getInstance()));
```

### Avoid Request Injection

```php
// Don't do this
$this->app->singleton(Service::class, fn ($app) => new Service($app['request']));

// Do this instead
$this->app->singleton(Service::class, fn ($app) => new Service(fn () => $app['request']));
```

## Concurrent Tasks (Swoole)

```php
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

## Ticks and Intervals (Swoole)

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10)
    ->immediate();
```

## Octane Cache (Swoole)

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

### Cache Intervals

```php
Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

## Tables (Swoole)

```php
// Configuration
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],

// Usage
Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/octane)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
