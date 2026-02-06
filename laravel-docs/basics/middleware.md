# Laravel 12.x Middleware Documentation

## Introduction

Middleware provides a convenient mechanism for inspecting and filtering HTTP requests entering your application. For example, Laravel includes middleware that verifies user authentication. If the user is not authenticated, the middleware redirects to the login screen; otherwise, it allows the request to proceed.

Additional middleware can be written to perform various tasks besides authentication, such as logging. User-defined middleware are typically located in the `app/Http/Middleware` directory.

## Defining Middleware

To create new middleware, use the `make:middleware` Artisan command:

```bash
php artisan make:middleware EnsureTokenIsValid
```

This creates a new middleware class in `app/Http/Middleware`. Here's an example middleware that validates a token:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

### Middleware and Responses

Middleware can execute logic **before** the request reaches the application:

```php
public function handle(Request $request, Closure $next): Response
{
    // Perform action before request

    return $next($request);
}
```

Or **after** the request is handled:

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    // Perform action after request

    return $response;
}
```

## Registering Middleware

### Global Middleware

To run middleware during every HTTP request, append it in `bootstrap/app.php`:

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```

Use `prepend()` to add middleware at the beginning of the list.

### Assigning Middleware to Routes

Assign middleware to specific routes:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

Assign multiple middleware using an array:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

#### Excluding Middleware

Exclude middleware from individual routes:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

Or exclude from an entire group:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

### Middleware Groups

Group several middleware under a single key in `bootstrap/app.php`:

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

Assign middleware groups to routes:

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

#### Laravel's Default Middleware Groups

**`web` Middleware Group:**
- `Illuminate\Cookie\Middleware\EncryptCookies`
- `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse`
- `Illuminate\Session\Middleware\StartSession`
- `Illuminate\View\Middleware\ShareErrorsFromSession`
- `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken`
- `Illuminate\Routing\Middleware\SubstituteBindings`

**`api` Middleware Group:**
- `Illuminate\Routing\Middleware\SubstituteBindings`

Modify default groups:

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

Replace middleware:

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

Remove middleware:

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

### Middleware Aliases

Define aliases for middleware in `bootstrap/app.php`:

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

Use the alias in routes:

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

#### Default Middleware Aliases

| Alias | Middleware |
|-------|-----------|
| `auth` | `Illuminate\Auth\Middleware\Authenticate` |
| `auth.basic` | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth` |
| `auth.session` | `Illuminate\Session\Middleware\AuthenticateSession` |
| `cache.headers` | `Illuminate\Http\Middleware\SetCacheHeaders` |
| `can` | `Illuminate\Auth\Middleware\Authorize` |
| `guest` | `Illuminate\Auth\Middleware\RedirectIfAuthenticated` |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword` |
| `signed` | `Illuminate\Routing\Middleware\ValidateSignature` |
| `throttle` | `Illuminate\Routing\Middleware\ThrottleRequests` |
| `verified` | `Illuminate\Auth\Middleware\EnsureEmailIsVerified` |

### Sorting Middleware

Specify middleware execution order using the `priority()` method:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

## Middleware Parameters

Middleware can receive additional parameters passed after the `$next` argument:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }
}
```

Pass parameters to middleware using a colon separator:

```php
use App\Http\Middleware\EnsureUserHasRole;

Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor');
```

Multiple parameters are delimited by commas:

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor,publisher');
```

## Terminable Middleware

Some middleware needs to perform work after the HTTP response is sent. Define a `terminate()` method, which is automatically called after the response is sent to the browser (when using FastCGI):

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

Register the terminable middleware in `bootstrap/app.php` with routes or global middleware.

To use the same middleware instance for both `handle()` and `terminate()` methods, register it as a singleton in your `AppServiceProvider`:

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/middleware)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
