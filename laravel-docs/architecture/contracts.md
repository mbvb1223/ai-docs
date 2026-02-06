# Laravel 12.x Contracts Documentation

## Introduction

Laravel's **contracts** are a set of interfaces that define the core services provided by the framework. Each contract has a corresponding implementation provided by the framework.

## Contracts vs. Facades

**Facades**: Simple to use, don't require constructor injection
**Contracts**: Allow explicit dependency definition, preferred for explicit dependency injection

Both can be used to create robust, well-tested Laravel applications.

## When to Use Contracts

Use contracts when:
- Building a package that integrates with multiple PHP frameworks
- Defining explicit dependencies without requiring Laravel's concrete implementations
- Your team prefers explicit dependency injection

## How to Use Contracts

Type-hint the interface in the constructor:

```php
<?php

namespace App\Listeners;

use App\Events\OrderWasPlaced;
use Illuminate\Contracts\Redis\Factory;

class CacheOrderInformation
{
    public function __construct(
        protected Factory $redis,
    ) {}

    public function handle(OrderWasPlaced $event): void
    {
        // ...
    }
}
```

## Contract Reference

| Contract | Facade |
|----------|--------|
| `Illuminate\Contracts\Auth\Factory` | `Auth` |
| `Illuminate\Contracts\Auth\Guard` | `Auth::guard()` |
| `Illuminate\Contracts\Auth\Access\Gate` | `Gate` |
| `Illuminate\Contracts\Auth\PasswordBroker` | `Password::broker()` |
| `Illuminate\Contracts\Broadcasting\Factory` | `Broadcast` |
| `Illuminate\Contracts\Bus\Dispatcher` | `Bus` |
| `Illuminate\Contracts\Cache\Factory` | `Cache` |
| `Illuminate\Contracts\Cache\Repository` | `Cache::driver()` |
| `Illuminate\Contracts\Config\Repository` | `Config` |
| `Illuminate\Contracts\Console\Kernel` | `Artisan` |
| `Illuminate\Contracts\Container\Container` | `App` |
| `Illuminate\Contracts\Cookie\Factory` | `Cookie` |
| `Illuminate\Contracts\Encryption\Encrypter` | `Crypt` |
| `Illuminate\Contracts\Events\Dispatcher` | `Event` |
| `Illuminate\Contracts\Filesystem\Factory` | `Storage` |
| `Illuminate\Contracts\Filesystem\Filesystem` | `Storage::disk()` |
| `Illuminate\Contracts\Hashing\Hasher` | `Hash` |
| `Illuminate\Contracts\Mail\Mailer` | `Mail` |
| `Illuminate\Contracts\Notifications\Dispatcher` | `Notification` |
| `Illuminate\Contracts\Queue\Factory` | `Queue` |
| `Illuminate\Contracts\Queue\Queue` | `Queue::connection()` |
| `Illuminate\Contracts\Redis\Factory` | `Redis` |
| `Illuminate\Contracts\Routing\Registrar` | `Route` |
| `Illuminate\Contracts\Routing\ResponseFactory` | `Response` |
| `Illuminate\Contracts\Routing\UrlGenerator` | `URL` |
| `Illuminate\Contracts\Session\Session` | `Session::driver()` |
| `Illuminate\Contracts\Translation\Translator` | `Lang` |
| `Illuminate\Contracts\Validation\Factory` | `Validator` |
| `Illuminate\Contracts\View\Factory` | `View` |

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/contracts)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
