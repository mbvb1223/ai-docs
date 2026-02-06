# Laravel Controllers Documentation

## Introduction

Controllers are classes that organize request handling logic instead of defining all logic as closures in route files. Controllers group related request handling into a single class. For example, a `UserController` might handle all requests related to users (showing, creating, updating, deleting).

By default, controllers are stored in the `app/Http/Controllers` directory.

## Writing Controllers

### Basic Controllers

Generate a new controller with the Artisan command:

```bash
php artisan make:controller UserController
```

Example basic controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Register the route:

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

**Note:** Controllers don't need to extend a base class, but it's sometimes convenient.

### Single Action Controllers

For complex single actions, dedicate an entire controller with an `__invoke` method:

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // ...
    }
}
```

Register without specifying a method:

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

Generate with `--invokable` option:

```bash
php artisan make:controller ProvisionServer --invokable
```

## Controller Middleware

### Middleware in Route Files

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

### Middleware in Controller Class

Implement the `HasMiddleware` interface:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    /**
     * Get the middleware that should be assigned to the controller.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }

    // ...
}
```

### Inline Middleware Closures

```php
use Closure;
use Illuminate\Http\Request;

public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```

## Resource Controllers

Resource controllers handle typical CRUD operations. Generate with:

```bash
php artisan make:controller PhotoController --resource
```

Register the resource route:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

Register multiple resources:

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

### Resource Controller Actions

| Verb | URI | Action | Route Name |
|------|-----|--------|-----------|
| GET | `/photos` | index | photos.index |
| GET | `/photos/create` | create | photos.create |
| POST | `/photos` | store | photos.store |
| GET | `/photos/{photo}` | show | photos.show |
| GET | `/photos/{photo}/edit` | edit | photos.edit |
| PUT/PATCH | `/photos/{photo}` | update | photos.update |
| DELETE | `/photos/{photo}` | destroy | photos.destroy |

### Partial Resource Routes

```php
use App\Http\Controllers\PhotoController;

// Only index and show
Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

// Exclude create, store, update, destroy
Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

### API Resource Routes

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

Or generate the controller:

```bash
php artisan make:controller PhotoController --api
```

### Nested Resources

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

This creates routes like `/photos/{photo}/comments/{comment}`

### Shallow Nesting

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

### Naming Resource Routes

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

### Naming Resource Route Parameters

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

Generates URI: `/users/{admin_user}`

### Scoping Resource Routes

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Creates URI: `/photos/{photo}/comments/{comment:slug}`

### Localizing Resource URIs

In `AppServiceProvider.php`:

```php
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

### Customizing Missing Model Behavior

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

### Soft Deleted Models

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();

// Or specific routes
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

### Singleton Resource Controllers

For resources with only one instance (like a user profile):

```php
use App\Http\Controllers\ProfileController;

Route::singleton('profile', ProfileController::class);
```

Routes registered:

| Verb | URI | Action | Route Name |
|------|-----|--------|-----------|
| GET | `/profile` | show | profile.show |
| GET | `/profile/edit` | edit | profile.edit |
| PUT/PATCH | `/profile` | update | profile.update |

### Creatable Singleton Resources

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

### Destroyable Singleton Resources

```php
Route::singleton(...)->destroyable();
```

### API Singleton Resources

```php
Route::apiSingleton('profile', ProfileController::class);

// With create/destroy routes
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

### Middleware and Resource Controllers

**Apply to all methods:**

```php
Route::resource('users', UserController::class)
    ->middleware(['auth', 'verified']);

Route::singleton('profile', ProfileController::class)
    ->middleware('auth');
```

**Apply to specific methods:**

```php
Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], 'auth');
```

**Exclude from specific methods:**

```php
Route::middleware(['auth', 'verified', 'subscribed'])->group(function () {
    Route::resource('users', UserController::class)
        ->withoutMiddlewareFor('index', ['auth', 'verified'])
        ->withoutMiddlewareFor(['create', 'store'], 'verified')
        ->withoutMiddlewareFor('destroy', 'subscribed');
});
```

## Dependency Injection and Controllers

### Constructor Injection

The service container automatically resolves dependencies:

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

### Method Injection

Type-hint dependencies in controller methods:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->name;

        // Store the user...

        return redirect('/users');
    }
}
```

**With route parameters:**

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

```php
public function update(Request $request, string $id): RedirectResponse
{
    // Update the user...

    return redirect('/users');
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/controllers)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
