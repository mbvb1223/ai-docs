# Laravel 12.x HTTP Session Documentation

## Introduction

Sessions provide a way to store user information across multiple HTTP requests. Laravel includes a variety of session backends accessed through a unified API, supporting Memcached, Redis, databases, and more.

## Configuration

Session configuration is stored in `config/session.php`. By default, Laravel uses the `database` session driver.

### Available Drivers

- **`file`** - Sessions stored in `storage/framework/sessions`
- **`cookie`** - Sessions stored in secure, encrypted cookies
- **`database`** - Sessions stored in a relational database
- **`memcached`/`redis`** - Fast, cache-based stores
- **`dynamodb`** - AWS DynamoDB storage
- **`array`** - PHP array storage (testing only, not persisted)

### Driver Prerequisites

#### Database
For the `database` driver, create the sessions table:
```bash
php artisan make:session-table
php artisan migrate
```

#### Redis
Install PhpRedis PHP extension via PECL or install `predis/predis` (~1.0) via Composer. Configure using the `SESSION_CONNECTION` environment variable or the `connection` option in `session.php`.

## Interacting With Sessions

### Retrieving Data

**Via Request Instance:**
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function show(Request $request, string $id)
    {
        $value = $request->session()->get('key');

        // With default value
        $value = $request->session()->get('key', 'default');

        // With closure as default
        $value = $request->session()->get('key', function () {
            return 'default';
        });
    }
}
```

**Via Global Session Helper:**
```php
Route::get('/home', function () {
    // Retrieve data
    $value = session('key');

    // With default
    $value = session('key', 'default');

    // Store data
    session(['key' => 'value']);
});
```

**Retrieve All Session Data:**
```php
$data = $request->session()->all();
```

**Retrieve Subset:**
```php
$data = $request->session()->only(['username', 'email']);
$data = $request->session()->except(['username', 'email']);
```

**Check Existence:**
```php
if ($request->session()->has('users')) {
    // Item exists and is not null
}

if ($request->session()->exists('users')) {
    // Item exists, even if null
}

if ($request->session()->missing('users')) {
    // Item does not exist
}
```

### Storing Data

```php
// Via request instance
$request->session()->put('key', 'value');

// Via global helper
session(['key' => 'value']);

// Push to array value
$request->session()->push('user.teams', 'developers');

// Retrieve and delete
$value = $request->session()->pull('key', 'default');

// Increment/Decrement
$request->session()->increment('count');
$request->session()->increment('count', $incrementBy = 2);
$request->session()->decrement('count');
$request->session()->decrement('count', $decrementBy = 2);
```

### Flash Data

Flash data is stored for the next request only:

```php
$request->session()->flash('status', 'Task was successful!');

// Persist flash data for additional request
$request->session()->reflash();

// Keep specific flash data
$request->session()->keep(['username', 'email']);

// Persist for current request only
$request->session()->now('status', 'Task was successful!');
```

### Deleting Data

```php
// Forget single key
$request->session()->forget('name');

// Forget multiple keys
$request->session()->forget(['name', 'status']);

// Flush all data
$request->session()->flush();
```

### Regenerating Session ID

Regenerate to prevent session fixation attacks:

```php
$request->session()->regenerate();

// Regenerate and remove all data
$request->session()->invalidate();
```

## Session Cache

Session cache provides user-specific caching scoped to individual sessions, supporting all cache methods (`get`, `put`, `remember`, `forget`, etc.):

```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->plus(minutes: 5)
);
```

Perfect for temporary, user-specific data like form data, temporary calculations, or API responses.

## Session Blocking

Prevents concurrent requests using the same session from executing simultaneously. Requires cache drivers supporting atomic locks: `memcached`, `dynamodb`, `redis`, `mongodb`, `database`, `file`, or `array`.

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

// With defaults (10 second lock, 10 second wait)
Route::post('/profile', function () {
    // ...
})->block();
```

Throws `Illuminate\Contracts\Cache\LockTimeoutException` if unable to obtain lock.

## Adding Custom Session Drivers

### Implementing the Driver

Create a class implementing `SessionHandlerInterface`:

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

**Method purposes:**
- **`open`** - Initialize (typically empty for most drivers)
- **`close`** - Cleanup (usually not needed)
- **`read`** - Return string version of session data for given ID
- **`write`** - Persist session data string to storage
- **`destroy`** - Remove session data for given ID
- **`gc`** - Destroy sessions older than given UNIX timestamp

### Registering the Driver

Register in a service provider's `boot` method:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Session::extend('mongo', function (Application $app) {
            return new MongoSessionHandler;
        });
    }
}
```

Configure using `SESSION_DRIVER=mongo` environment variable or in `config/session.php`.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/session)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
