# Laravel 12.x Sanctum Documentation

## Introduction

Laravel Sanctum provides authentication for SPAs, mobile applications, and token-based APIs.

## Installation

```bash
php artisan install:api
```

## API Token Authentication

### Setup

Add the `HasApiTokens` trait to your User model:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

### Issuing Tokens

```php
$token = $request->user()->createToken($request->token_name);
return ['token' => $token->plainTextToken];
```

### Token Abilities

```php
// Create with abilities
$token = $user->createToken('token-name', ['server:update'])->plainTextToken;

// Check abilities
if ($user->tokenCan('server:update')) {
    // ...
}
```

### Protecting Routes

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

### Revoking Tokens

```php
$user->tokens()->delete();
$request->user()->currentAccessToken()->delete();
```

### Token Expiration

```php
// In config
'expiration' => 525600,

// Per token
$user->createToken('token-name', ['*'], now()->plus(weeks: 1));
```

## SPA Authentication

### Configuration

Set stateful domains in `sanctum` config and enable middleware:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->statefulApi();
})
```

### CORS Configuration

```php
'supports_credentials' => true,
```

### Authenticating

Initialize CSRF protection:

```javascript
axios.get('/sanctum/csrf-cookie').then(response => {
    // Login...
});
```

## Mobile Authentication

### Issuing Tokens

```php
Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

## Testing

```php
use Laravel\Sanctum\Sanctum;

Sanctum::actingAs(
    User::factory()->create(),
    ['view-tasks']
);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/sanctum)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
