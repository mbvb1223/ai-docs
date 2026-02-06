# Laravel 12.x Authorization Documentation

## Introduction

Laravel provides two primary ways to authorize user actions: **Gates** and **Policies**. Gates offer simple, closure-based authorization, while policies organize authorization logic around specific models or resources.

## Gates

### Writing Gates

Gates are closures that determine if a user is authorized to perform a given action. Define them in the `boot` method of `AppServiceProvider`:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

You can also use class callback arrays:

```php
Gate::define('update-post', [PostPolicy::class, 'update']);
```

### Authorizing Actions via Gates

Use the `allows` or `denies` methods:

```php
if (! Gate::allows('update-post', $post)) {
    abort(403);
}

if (Gate::forUser($user)->allows('update-post', $post)) {
    // The user can update the post...
}
```

**Check multiple actions:**

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // User can update OR delete the post...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // User can't update OR delete the post...
}
```

**Authorize or throw exception:**

```php
Gate::authorize('update-post', $post);
// The action is authorized...
```

### Gate Responses

Return detailed responses with error messages:

```php
use Illuminate\Auth\Access\Response;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('You must be an administrator.');
});
```

Inspect responses:

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // Authorized
} else {
    echo $response->message();
}
```

**Custom HTTP status codes:**

```php
Response::denyWithStatus(404);
Response::denyAsNotFound(); // Convenience method for 404
```

### Intercepting Gate Checks

Use `before` and `after` methods:

```php
Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

### Inline Authorization

```php
Gate::allowIf(fn (User $user) => $user->isAdministrator());
Gate::denyIf(fn (User $user) => $user->banned());
```

## Creating Policies

### Generating Policies

```bash
php artisan make:policy PostPolicy
php artisan make:policy PostPolicy --model=Post
```

### Registering Policies

**Policy Discovery (Automatic):**
Laravel automatically discovers policies if they follow naming conventions:
- Model: `App\Models\Post`
- Policy: `App\Policies\PostPolicy`

**Custom Discovery:**

```php
Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // Return policy class name for the given model
});
```

**Manual Registration:**

```php
Gate::policy(Order::class, OrderPolicy::class);
```

**Using Attributes:**

```php
use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

## Writing Policies

### Policy Methods

```php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

Standard methods generated with `--model`: `viewAny`, `view`, `create`, `update`, `delete`, `restore`, `forceDelete`.

### Policy Responses

Return `Illuminate\Auth\Access\Response` for detailed messages:

```php
use Illuminate\Auth\Access\Response;

public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('You do not own this post.');
}
```

**Custom HTTP status:**

```php
Response::denyWithStatus(404);
Response::denyAsNotFound();
```

### Methods Without Models

For actions that don't require a model instance (e.g., `create`):

```php
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

### Guest Users

Allow guest users by making the user parameter optional:

```php
public function update(?User $user, Post $post): bool
{
    return $user?->id === $post->user_id;
}
```

### Policy Filters

Use `before` method to pre-authorize actions:

```php
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

## Authorizing Actions Using Policies

### Via the User Model

```php
if ($request->user()->cannot('update', $post)) {
    abort(403);
}

if ($request->user()->can('update', $post)) {
    // User can update
}
```

**For actions without models:**

```php
if ($request->user()->cannot('create', Post::class)) {
    abort(403);
}
```

### Via the Gate Facade

```php
use Illuminate\Support\Facades\Gate;

Gate::authorize('update', $post);
// The action is authorized...
```

**For actions without models:**

```php
Gate::authorize('create', Post::class);
```

### Via Middleware

Attach authorization using the `can` middleware alias:

```php
Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
```

Using the `can` method:

```php
Route::put('/post/{post}', function (Post $post) {
    // ...
})->can('update', 'post');
```

**For actions without models:**

```php
Route::post('/post', function () {
    // The current user may create posts...
})->can('create', Post::class);
```

### Via Blade Templates

```blade
@can('update', $post)
    <!-- The current user can update the post... -->
@elsecan('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- The current user cannot update the post... -->
@elsecannot('create', App\Models\Post::class)
    <!-- The current user cannot create new posts... -->
@endcannot
```

**Check multiple actions:**

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- The current user can update, view, or delete the post... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- The current user can create a post... -->
@endcanany
```

### Supplying Additional Context

Pass an array as the second argument to include additional parameters:

```php
Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // The user can create the post...
}
```

**In policy methods:**

```php
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}

Gate::authorize('update', [$post, $request->category]);
```

## Authorization & Inertia

Share authorization data in the `HandleInertiaRequests` middleware:

```php
public function share(Request $request)
{
    return [
        ...parent::share($request),
        'auth' => [
            'user' => $request->user(),
            'permissions' => [
                'post' => [
                    'create' => $request->user()->can('create', Post::class),
                ],
            ],
        ],
    ];
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/authorization)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
