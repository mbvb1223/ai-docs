# Laravel 12.x Error Handling Documentation

## Introduction

Error and exception handling is pre-configured in Laravel. Manage exceptions using the `withExceptions` method in `bootstrap/app.php`.

## Configuration

The `debug` option in `config/app.php` determines error detail display. Set `APP_DEBUG=true` only during development; use `false` in production.

## Handling Exceptions

### Reporting Exceptions

Register custom reporting in `bootstrap/app.php`:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // Custom reporting logic
    });
})
```

**Stop default logging:**

```php
$exceptions->report(function (InvalidOrderException $e) {
    // ...
})->stop();
```

#### Global Log Context

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

#### The `report()` Helper

```php
public function isValid(string $value): bool
{
    try {
        // Validate the value...
    } catch (Throwable $e) {
        report($e);
        return false;
    }
}
```

### Exception Log Levels

```php
use PDOException;
use Psr\Log\LogLevel;

$exceptions->level(PDOException::class, LogLevel::CRITICAL);
```

### Ignoring Exceptions by Type

```php
use App\Exceptions\InvalidOrderException;

$exceptions->dontReport([
    InvalidOrderException::class,
]);
```

### Rendering Exceptions

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

$exceptions->render(function (InvalidOrderException $e, Request $request) {
    return response()->view('errors.invalid-order', status: 500);
});
```

#### Rendering Exceptions as JSON

```php
$exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
    return $request->is('admin/*') || $request->expectsJson();
});
```

### Reportable and Renderable Exceptions

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    public function report(): void
    {
        // ...
    }

    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

## Throttling Reported Exceptions

### Using Lottery

```php
use Illuminate\Support\Lottery;

$exceptions->throttle(function (Throwable $e) {
    return Lottery::odds(1, 1000);
});
```

### Rate Limiting

```php
use Illuminate\Cache\RateLimiting\Limit;

$exceptions->throttle(function (Throwable $e) {
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300);
    }
});
```

## HTTP Exceptions

```php
abort(404);
```

### Custom HTTP Error Pages

Create views in `resources/views/errors/` named after status codes (e.g., `404.blade.php`):

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

**Publish default templates:**

```bash
php artisan vendor:publish --tag=laravel-errors
```

### Fallback HTTP Error Pages

Create fallback templates:
- `resources/views/errors/4xx.blade.php`
- `resources/views/errors/5xx.blade.php`

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/errors)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
