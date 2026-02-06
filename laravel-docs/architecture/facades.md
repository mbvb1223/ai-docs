# Laravel 12.x Facades Documentation

## Introduction

Facades provide a "static" interface to classes available in the application's [service container](container.md). They serve as "static proxies" to underlying classes, providing a terse, expressive syntax while maintaining testability and flexibility.

All Laravel facades are defined in the `Illuminate\Support\Facades` namespace.

### Basic Usage

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

### Helper Functions

Laravel also offers global "helper functions" that complement facades for easier interaction with common features:

```php
// Using facade
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

// Using helper function
Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

---

## When to Utilize Facades

### Benefits
- **Terse, memorable syntax** - Easy to use without remembering long class names
- **Easy to test** - Due to unique usage of PHP's dynamic methods

### Caution: Scope Creep
Pay special attention to class size when using facades. If a class is getting too large, consider splitting it into multiple smaller classes.

### Facades vs. Dependency Injection

Facades can be tested just like injected class instances:

**Pest/PHPUnit Example:**

```php
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

### Facades vs. Helper Functions

Facades and helper functions are equivalent in practice and can be tested identically:

```php
// Both are equivalent
Route::get('/cache', function () {
    return cache('key');
});
```

---

## How Facades Work

Facades extend the base `Illuminate\Support\Facades\Facade` class and use PHP's `__callStatic()` magic method to defer calls to service container objects.

### Example: Cache Facade Implementation

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

### How It Works

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

When `Cache::get('user:1')` is called:
1. Laravel resolves the `cache` binding from the service container
2. The `get` method is executed on that object
3. The result is returned

---

## Real-Time Facades

Real-time facades allow you to treat any class as if it were a facade by prefixing its namespace with `Facades`.

### Without Real-Time Facades

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

### With Real-Time Facades

```php
<?php

namespace App\Models;

use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(): void
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this);
    }
}
```

### Testing Real-Time Facades

**Pest:**

```php
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

**PHPUnit:**

```php
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

---

## Facade Class Reference

| Facade | Class | Service Container Binding |
|--------|-------|---------------------------|
| `App` | `Illuminate\Foundation\Application` | `app` |
| `Artisan` | `Illuminate\Contracts\Console\Kernel` | `artisan` |
| `Auth` | `Illuminate\Auth\AuthManager` | `auth` |
| `Blade` | `Illuminate\View\Compilers\BladeCompiler` | `blade.compiler` |
| `Cache` | `Illuminate\Cache\CacheManager` | `cache` |
| `Config` | `Illuminate\Config\Repository` | `config` |
| `Cookie` | `Illuminate\Cookie\CookieJar` | `cookie` |
| `Crypt` | `Illuminate\Encryption\Encrypter` | `encrypter` |
| `DB` | `Illuminate\Database\DatabaseManager` | `db` |
| `Event` | `Illuminate\Events\Dispatcher` | `events` |
| `File` | `Illuminate\Filesystem\Filesystem` | `files` |
| `Hash` | `Illuminate\Contracts\Hashing\Hasher` | `hash` |
| `Log` | `Illuminate\Log\LogManager` | `log` |
| `Mail` | `Illuminate\Mail\Mailer` | `mailer` |
| `Queue` | `Illuminate\Queue\QueueManager` | `queue` |
| `Redirect` | `Illuminate\Routing\Redirector` | `redirect` |
| `Route` | `Illuminate\Routing\Router` | `router` |
| `Storage` | `Illuminate\Filesystem\FilesystemManager` | `filesystem` |
| `URL` | `Illuminate\Routing\UrlGenerator` | `url` |
| `View` | `Illuminate\View\Factory` | `view` |

*See the full API documentation at [api.laravel.com/docs/12.x](https://api.laravel.com/docs/12.x)*

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/facades)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
