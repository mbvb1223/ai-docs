# Laravel 12.x Authentication Documentation

## Introduction

Laravel's authentication system is built on two core concepts:

- **Guards**: Define how users are authenticated for each request (e.g., `session` guard uses session storage and cookies)
- **Providers**: Define how users are retrieved from persistent storage (Eloquent or database query builder)

Authentication configuration is located at `config/auth.php`.

### Starter Kits

Laravel provides [application starter kits](../getting-started/starter-kits.md) that scaffold the entire authentication system automatically.

### Database Considerations

- Default model: `App\Models\User` (Eloquent)
- Password column must be at least 60 characters
- Users table requires a nullable `remember_token` column (100 characters)

---

## Authentication Quickstart

### Retrieving the Authenticated User

```php
use Illuminate\Support\Facades\Auth;

// Retrieve the currently authenticated user
$user = Auth::user();

// Retrieve the user's ID
$id = Auth::id();
```

Or via request injection:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FlightController extends Controller
{
    public function update(Request $request)
    {
        $user = $request->user();
        // ...
    }
}
```

### Determining if User is Authenticated

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // User is logged in
}
```

### Protecting Routes

Use the `auth` middleware:

```php
Route::get('/flights', function () {
    // Only authenticated users may access this route
})->middleware('auth');
```

**Redirect Unauthenticated Users** (`bootstrap/app.php`):

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');

    // Using a closure
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

**Specifying a Guard**:

```php
Route::get('/flights', function () {
    // ...
})->middleware('auth:admin');
```

---

## Manually Authenticating Users

### Basic Authentication

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function authenticate(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

**Important**: Do not hash the password—Laravel does this automatically.

### Specifying Additional Conditions

```php
// Simple conditions
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // Authentication successful
}

// Complex conditions with closure
use Illuminate\Database\Eloquent\Builder;

if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // Authentication successful
}
```

### Accessing Specific Guard Instances

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

### Remembering Users

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // User is being remembered
}

// Check if user was authenticated via "remember me" cookie
if (Auth::viaRemember()) {
    // ...
}
```

### Other Authentication Methods

**Authenticate a User Instance**:

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);

// With "remember me"
Auth::login($user, $remember = true);

// Specific guard
Auth::guard('admin')->login($user);
```

**Authenticate by ID**:

```php
Auth::loginUsingId(1);

// With "remember me"
Auth::loginUsingId(1, remember: true);
```

**One-time Authentication** (no sessions/cookies):

```php
if (Auth::once($credentials)) {
    // ...
}
```

---

## HTTP Basic Authentication

```php
Route::get('/profile', function () {
    // Only authenticated users may access this route
})->middleware('auth.basic');
```

### Stateless HTTP Basic Authentication

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;

class AuthenticateOnceWithBasicAuth
{
    public function handle(Request $request, Closure $next): Response
    {
        return Auth::onceBasic() ?: $next($request);
    }
}
```

---

## Logging Out

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

public function logout(Request $request): RedirectResponse
{
    Auth::logout();

    $request->session()->invalidate();
    $request->session()->regenerateToken();

    return redirect('/');
}
```

### Invalidating Sessions on Other Devices

Add `auth.session` middleware to routes:

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

Then logout other devices:

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

---

## Password Confirmation

### Configuration

Configure timeout in `config/auth.php`:

```php
'password_timeout' => 10800, // 3 hours (default)
```

### Protecting Routes

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

---

## Adding Custom Guards

Define in a service provider:

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Auth::extend('jwt', function (Application $app, string $name, array $config) {
            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

Configure in `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

### Closure Request Guards

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

---

## Authentication Events

Laravel dispatches the following events:

| Event | Description |
|-------|-------------|
| `Registered` | User registered |
| `Attempting` | Login attempt |
| `Authenticated` | User authenticated |
| `Login` | User logged in |
| `Failed` | Authentication failed |
| `Validated` | Credentials validated |
| `Verified` | Email verified |
| `Logout` | User logged out |
| `CurrentDeviceLogout` | Current device logged out |
| `OtherDeviceLogout` | Other device logged out |
| `Lockout` | User locked out |
| `PasswordReset` | Password reset |
| `PasswordResetLinkSent` | Reset link sent |

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/authentication)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
