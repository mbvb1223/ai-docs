# Laravel 12.x Cache Documentation

## Introduction

Laravel provides an expressive, unified API for various cache backends. Cache stores retrieved data in fast data stores like Memcached or Redis to improve web application performance.

## Configuration

Cache configuration is in `config/cache.php`. Laravel supports:
- **Memcached**
- **Redis**
- **DynamoDB**
- **Relational Databases**
- **File-based cache**
- **Array and null drivers** (for testing)

Default: `database` cache driver

### Driver Prerequisites

#### Database
Create cache table using:
```bash
php artisan make:cache-table
php artisan migrate
```

#### Memcached
Requires Memcached PECL package. Configuration example:
```php
'memcached' => [
    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

#### Redis
Requires PhpRedis extension or `predis/predis` package (~2.0)

#### DynamoDB
Requires AWS SDK and TTL configuration for automatic expiration cleanup.

#### MongoDB
Use `mongodb/laravel-mongodb` package with TTL index support.

## Cache Usage

### Obtaining a Cache Instance

Using the `Cache` facade:
```php
<?php
namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    public function index(): array
    {
        $value = Cache::get('key');
        return [/* ... */];
    }
}
```

### Accessing Multiple Cache Stores
```php
$value = Cache::store('file')->get('foo');
Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

### Retrieving Items From the Cache

**Basic retrieval:**
```php
$value = Cache::get('key');
$value = Cache::get('key', 'default');
```

**With closure default:**
```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

**Check existence:**
```php
if (Cache::has('key')) {
    // ...
}
```

**Increment/Decrement:**
```php
Cache::add('key', 0, now()->plus(hours: 4));
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

**Retrieve and Store:**
```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});

$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

**Stale While Revalidate Pattern:**
```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```
- First value (5): fresh period
- Second value (10): stale period

**Retrieve and Delete:**
```php
$value = Cache::pull('key');
$value = Cache::pull('key', 'default');
```

### Storing Items in the Cache

**Basic storage:**
```php
Cache::put('key', 'value', $seconds = 10);
Cache::put('key', 'value'); // Indefinite
Cache::put('key', 'value', now()->plus(minutes: 10));
```

**Store if Not Present:**
```php
Cache::add('key', 'value', $seconds);
```
Returns `true` if added, `false` if already exists (atomic operation).

**Store Forever:**
```php
Cache::forever('key', 'value');
```
Must be manually removed with `forget()`.

### Removing Items From the Cache

**Forget single item:**
```php
Cache::forget('key');
```

**Remove via expiration:**
```php
Cache::put('key', 'value', 0);
Cache::put('key', 'value', -5);
```

**Flush entire cache:**
```php
Cache::flush();
```

### Cache Memoization

Temporarily store resolved cache values in memory during a single request/job:

```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');

// Using specific store
$value = Cache::memo('redis')->get('key');
```

First `get()` hits cache; subsequent calls return memoized value.

### The Cache Helper

**Retrieve value:**
```php
$value = cache('key');
```

**Store values:**
```php
cache(['key' => 'value'], $seconds);
cache(['key' => 'value'], now()->plus(minutes: 10));
```

**Access factory:**
```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

## Cache Tags

**Note:** Not supported with `file`, `dynamodb`, or `database` drivers.

**Storing tagged items:**
```php
use Illuminate\Support\Facades\Cache;

Cache::tags(['people', 'artists'])->put('John', $john, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

**Accessing tagged items:**
```php
$john = Cache::tags(['people', 'artists'])->get('John');
$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

**Removing tagged items:**
```php
// Removes items tagged with 'people', 'authors', or both
Cache::tags(['people', 'authors'])->flush();

// Removes only items tagged with 'authors'
Cache::tags('authors')->flush();
```

## Atomic Locks

Manipulate distributed locks without race conditions. Available drivers: `memcached`, `redis`, `dynamodb`, `database`, `file`, `array`.

### Managing Locks

**Basic lock:**
```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...
    $lock->release();
}
```

**With closure (auto-release):**
```php
Cache::lock('foo', 10)->get(function () {
    // Lock acquired for 10 seconds and automatically released...
});
```

**Blocking with timeout:**
```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);
    // Lock acquired after waiting max 5 seconds...
} catch (LockTimeoutException $e) {
    // Unable to acquire lock...
} finally {
    $lock->release();
}
```

**Blocking with closure:**
```php
Cache::lock('foo', 10)->block(5, function () {
    // Lock acquired for 10 seconds after waiting max 5 seconds...
});
```

### Managing Locks Across Processes

**Acquire in one process, release in another:**
```php
$podcast = Podcast::find($id);
$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

**Restore and release in job:**
```php
Cache::restoreLock('processing', $this->owner)->release();
```

**Force release:**
```php
Cache::lock('processing')->forceRelease();
```

### Locks and Function Invocations

**Without overlapping:**
```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired after waiting max 10 seconds...
});
```

**Customize lock duration and wait time:**
```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired for 120 seconds after waiting max 5 seconds...
}, lockFor: 120, waitFor: 5);
```

## Cache Failover

Automatic failover to next configured store if primary fails:

```php
'failover' => [
    'driver' => 'failover',
    'stores' => [
        'database',
        'array',
    ],
],
```

**Enable failover:**
```
CACHE_STORE=failover
```

Dispatches `Illuminate\Cache\Events\CacheFailedOver` event on failure.

## Adding Custom Cache Drivers

### Writing the Driver

Implement `Illuminate\Contracts\Cache\Store`:

```php
<?php
namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

### Registering the Driver

Register in `AppServiceProvider`:

```php
<?php
namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }
}
```

Update `CACHE_STORE` environment variable to use the new driver.

## Events

Cache events available:
- `CacheFlushed`
- `CacheFlushing`
- `CacheHit`
- `CacheMissed`
- `ForgettingKey`
- `KeyForgetFailed`
- `KeyForgotten`
- `KeyWriteFailed`
- `KeyWritten`
- `RetrievingKey`
- `RetrievingManyKeys`
- `WritingKey`
- `WritingManyKeys`

**Disable events for performance:**

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/cache)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
