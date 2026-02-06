# Laravel 12.x Routing Documentation

## Overview

Laravel routing provides an expressive method of defining routes and their behavior. Routes are defined in files located in the `routes` directory and are automatically loaded via `bootstrap/app.php`.

## Basic Routing

### Simple Route Definition

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

### Default Route Files

- **`routes/web.php`** - Web interface routes (includes session state and CSRF protection)
- **`routes/api.php`** - Stateless API routes (requires `php artisan install:api`)

### Available HTTP Methods

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);

// Multiple methods
Route::match(['get', 'post'], '/', function () {});

// All HTTP verbs
Route::any('/', function () {});
```

### Dependency Injection

```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // Request is automatically injected
});
```

### CSRF Protection

All `POST`, `PUT`, `PATCH`, or `DELETE` routes in `routes/web.php` require CSRF tokens:

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

## Special Route Types

### Redirect Routes

```php
Route::redirect('/here', '/there');
Route::redirect('/here', '/there', 301);
Route::permanentRedirect('/here', '/there');
```

### View Routes

```php
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

### Listing Routes

```bash
php artisan route:list
php artisan route:list -v              # Include middleware
php artisan route:list -vv             # Expand middleware groups
php artisan route:list --path=api      # Filter by path
php artisan route:list --except-vendor # Hide third-party routes
php artisan route:list --only-vendor   # Show only third-party routes
```

## Route Parameters

### Required Parameters

```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});

Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

**Note:** Parameter names don't matter for injection order; they're based on position.

### Optional Parameters

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

### Regular Expression Constraints

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

// Multiple constraints
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

### Convenient Constraint Methods

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);
```

### Global Constraints

Define in `App\Providers\AppServiceProvider`:

```php
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

### Encoded Forward Slashes

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

## Named Routes

### Creating Named Routes

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');

// With controller
Route::get('/user/profile', [UserProfileController::class, 'show'])->name('profile');
```

### Generating URLs to Named Routes

```php
$url = route('profile');

return redirect()->route('profile');
return to_route('profile');

// With parameters
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
$url = route('profile', ['id' => 1, 'photos' => 'yes']); // Query string added
```

### Inspecting Current Route

```php
if ($request->route()->named('profile')) {
    // ...
}
```

## Route Groups

### Middleware Groups

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware
    });

    Route::get('/user/profile', function () {
        // Uses first & second middleware
    });
});
```

### Controller Groups

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

### Subdomain Routing

```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

### Route Prefixes

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches "/admin/users"
    });
});
```

### Route Name Prefixes

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"
    })->name('users');
});
```

## Route Model Binding

### Implicit Binding

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});

// With controller
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

**Note:** Variable name `{user}` must match the model type-hint.

### Soft Deleted Models

```php
Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

### Customizing the Key

```php
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});

// Override on model
public function getRouteKeyName(): string
{
    return 'slug';
}
```

### Scoped Binding

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});

// Explicit scoping
Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();

// Group scoping
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});

// Disable scoping
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

### Customizing Missing Model Behavior

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

### Implicit Enum Binding

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}

// Route
use App\Enums\Category;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

### Explicit Binding

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::model('user', User::class);
}

// Usage
Route::get('/users/{user}', function (User $user) {
    // ...
});
```

### Custom Resolution Logic

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}

// Or override on model
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}

// For child bindings
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

## Fallback Routes

```php
Route::fallback(function () {
    // Handle unmatched routes
});
```

## Rate Limiting

### Defining Rate Limiters

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

### Attaching Rate Limiters to Routes

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});
```

## Form Method Spoofing

HTML forms don't support `PUT`, `PATCH`, or `DELETE`. Use the `_method` field:

```html
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>

<!-- Or use Blade directive -->
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

## Accessing the Current Route

```php
use Illuminate\Support\Facades\Route;

$route = Route::current();                    // Illuminate\Routing\Route
$name = Route::currentRouteName();            // string
$action = Route::currentRouteAction();        // string
```

## CORS (Cross-Origin Resource Sharing)

Automatically handled by the `HandleCors` middleware. Customize via:

```bash
php artisan config:publish cors
```

## Route Caching

### Generate Cache

```bash
php artisan route:cache
```

Drastically improves performance in production by caching all registered routes.

### Clear Cache

```bash
php artisan route:clear
```

Run `route:cache` during deployment whenever routes are added or modified.

## Routing Customization

Customize route registration in `bootstrap/app.php`:

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/routing)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
