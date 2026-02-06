# Laravel 12.x HTTP Requests Documentation

## Introduction

Laravel's `Illuminate\Http\Request` class provides an object-oriented way to interact with the current HTTP request and retrieve input, cookies, and files.

## Accessing the Request

### Via Dependency Injection

```php
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');
        return redirect('/users');
    }
}
```

### Via Route Closures

```php
Route::get('/', function (Request $request) {
    // ...
});
```

## Request Path, Host, and Method

```php
$uri = $request->path();  // 'foo/bar'
$url = $request->url();
$urlWithQueryString = $request->fullUrl();

if ($request->is('admin/*')) { }
if ($request->routeIs('admin.*')) { }

$method = $request->method();
if ($request->isMethod('post')) { }
```

## Request Headers

```php
$value = $request->header('X-Header-Name');
$value = $request->header('X-Header-Name', 'default');
$token = $request->bearerToken();

if ($request->hasHeader('X-Header-Name')) { }
```

## Retrieving Input

### All Input Data

```php
$input = $request->all();
$input = $request->collect();
```

### Single Input Value

```php
$name = $request->input('name');
$name = $request->input('name', 'default');
$name = $request->input('products.0.name');  // Nested
$names = $request->input('products.*.name');
```

### Query String Input

```php
$name = $request->query('name');
$query = $request->query();
```

### Typed Input Values

```php
$name = $request->string('name')->trim();
$perPage = $request->integer('per_page');
$archived = $request->boolean('archived');
$versions = $request->array('versions');
$birthday = $request->date('birthday');
$status = $request->enum('status', Status::class);
```

### Partial Input

```php
$input = $request->only(['username', 'password']);
$input = $request->except(['credit_card']);
```

## Input Presence

```php
if ($request->has('name')) { }
if ($request->has(['name', 'email'])) { }
if ($request->hasAny(['name', 'email'])) { }
if ($request->filled('name')) { }
if ($request->missing('name')) { }

$request->whenHas('name', function (string $input) {
    // ...
});

$request->whenFilled('name', function (string $input) {
    // ...
});
```

## Merging Additional Input

```php
$request->merge(['votes' => 0]);
$request->mergeIfMissing(['votes' => 0]);
```

## Old Input

```php
$request->flash();
$request->flashOnly(['username', 'email']);
$request->flashExcept('password');

return redirect('/form')->withInput();
$username = $request->old('username');
```

In Blade:

```blade
<input type="text" name="username" value="{{ old('username') }}">
```

## Cookies

```php
$value = $request->cookie('name');
```

## Files

```php
$file = $request->file('photo');
$file = $request->photo;

if ($request->hasFile('photo')) { }
if ($request->file('photo')->isValid()) { }

$path = $request->photo->path();
$extension = $request->photo->extension();
$path = $request->photo->store('images');
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## Configuring Trusted Proxies

In `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

## Configuring Trusted Hosts

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$']);
})
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/requests)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
