# Service Providers - Laravel 12.x Documentation

## Introduction

Service providers are the central place of all Laravel application bootstrapping. They handle:
- **Registering** service container bindings
- **Registering** event listeners
- **Registering** middleware
- **Registering** routes

All user-defined service providers are registered in the `bootstrap/providers.php` file.

Laravel uses dozens of service providers internally to bootstrap core services (mailer, queue, cache, etc.). Many are "deferred" providers, loaded only when needed.

## Writing Service Providers

All service providers extend `Illuminate\Support\ServiceProvider` and contain two main methods: `register` and `boot`.

### The Register Method

**Only bind things into the service container** — never register event listeners, routes, or other functionality here, as you may accidentally use services from providers not yet loaded.

#### Basic Example

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

#### Using `bindings` and `singletons` Properties

For many simple bindings, use the `bindings` and `singletons` properties to auto-register:

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

### The Boot Method

Called **after all other service providers have been registered**, so you can access any services they provided. Use this for registering view composers, event listeners, and other functionality.

#### Basic Example

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

#### Boot Method Dependency Injection

Type-hint dependencies in the `boot` method — the service container will automatically inject them:

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Bootstrap any application services.
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

## Registering Providers

All service providers are registered in `bootstrap/providers.php`:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

Generate new providers with:

```bash
php artisan make:provider RiakServiceProvider
```

Laravel automatically adds the generated provider to `bootstrap/providers.php`. Manually created providers must be added manually:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class,
];
```

## Deferred Providers

If your provider **only registers service container bindings**, defer its registration to improve performance. It will only load when one of its bindings is actually needed.

Implement `\Illuminate\Contracts\Support\DeferrableProvider` and define a `provides` method:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```

The `provides` method returns an array of service container bindings registered by the provider.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/providers)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
