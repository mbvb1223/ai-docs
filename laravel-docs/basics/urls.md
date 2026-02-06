# Laravel 12.x URL Generation Documentation

## Introduction

Laravel provides several helpers to generate URLs for your application, useful for building links in templates, API responses, and redirect responses.

## Generating URLs

### Basic URL Generation

```php
$post = App\Models\Post::find(1);
echo url("/posts/{$post->id}");
// http://example.com/posts/1
```

### With Query Parameters

```php
echo url()->query('/posts', ['search' => 'Laravel']);
// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);
// http://example.com/posts?sort=latest&search=Laravel
```

## Accessing Current URL

```php
echo url()->current();   // Without query string
echo url()->full();      // With query string

use Illuminate\Support\Facades\URL;
echo URL::current();
```

## Previous URL

```php
echo url()->previous();
echo url()->previousPath();
$previousRoute = $request->session()->previousRoute();
```

## URLs for Named Routes

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');

echo route('post.show', ['post' => 1]);
// http://example.com/post/1

// Multiple parameters
echo route('comment.show', ['post' => 1, 'comment' => 3]);

// Additional parameters become query string
echo route('post.show', ['post' => 1, 'search' => 'rocket']);
// http://example.com/post/1?search=rocket

// Eloquent models
echo route('post.show', ['post' => $post]);
```

## Signed URLs

Create URLs with a signature hash to prevent modification:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);

// Temporary signed URLs
return URL::temporarySignedRoute(
    'unsubscribe', now()->plus(minutes: 30), ['user' => 1]
);
```

### Validating Signed Routes

```php
Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }
})->name('unsubscribe');

// Using middleware
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->middleware('signed');
```

## URLs for Controller Actions

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

## Fluent URI Objects

```php
use Illuminate\Support\Uri;

$uri = Uri::of('https://example.com/path');
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::action([UserController::class, 'index']);

// Fluent modification
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

## Default URL Values

Set request-wide default values via middleware:

```php
namespace App\Http\Middleware;

use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);
        return $next($request);
    }
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/urls)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
