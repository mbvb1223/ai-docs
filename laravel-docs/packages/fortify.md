# Laravel 12.x Fortify Documentation

## Introduction

Laravel Fortify is a frontend-agnostic authentication backend implementation for Laravel. It provides registration, login, password reset, email verification, and two-factor authentication without a built-in UI.

## Installation

```bash
composer require laravel/fortify
php artisan fortify:install
php artisan migrate
```

## Configuration

Enable features in `config/fortify.php`:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

For SPAs, disable view routes:

```php
'views' => false,
```

## Authentication

### Customizing Login View

```php
use Laravel\Fortify\Fortify;

public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });
}
```

### Custom Authentication Logic

```php
Fortify::authenticateUsing(function (Request $request) {
    $user = User::where('email', $request->email)->first();

    if ($user && Hash::check($request->password, $user->password)) {
        return $user;
    }
});
```

### Custom Authentication Pipeline

```php
Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
        EnsureLoginIsNotThrottled::class,
        CanonicalizeUsername::class,
        RedirectIfTwoFactorAuthenticatable::class,
        AttemptToAuthenticate::class,
        PrepareAuthenticatedSession::class,
    ]);
});
```

## Two-Factor Authentication

### Setup

Add trait to User model:

```php
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use TwoFactorAuthenticatable;
}
```

Enable in config:

```php
Features::twoFactorAuthentication(),
```

### Enabling 2FA

POST to `/user/two-factor-authentication`

### QR Code

```php
$request->user()->twoFactorQrCodeSvg();
```

Or GET `/user/two-factor-qr-code` for JSON with `svg` key.

### Recovery Codes

```php
(array) $request->user()->recoveryCodes()
```

Or GET `/user/two-factor-recovery-codes`

### Challenge View

```php
Fortify::twoFactorChallengeView(function () {
    return view('auth.two-factor-challenge');
});
```

POST to `/two-factor-challenge` with `code` or `recovery_code`.

## Registration

```php
Fortify::registerView(function () {
    return view('auth.register');
});
```

POST to `/register` with `name`, `email`, `password`, `password_confirmation`.

Customize in `App\Actions\Fortify\CreateNewUser`.

## Password Reset

### Request Link

```php
Fortify::requestPasswordResetLinkView(function () {
    return view('auth.forgot-password');
});
```

POST to `/forgot-password` with `email`.

### Reset Password

```php
Fortify::resetPasswordView(function (Request $request) {
    return view('auth.reset-password', ['request' => $request]);
});
```

POST to `/reset-password` with `email`, `password`, `password_confirmation`, `token`.

## Email Verification

Ensure User implements `MustVerifyEmail`:

```php
Features::emailVerification(),
```

```php
Fortify::verifyEmailView(function () {
    return view('auth.verify-email');
});
```

POST to `/email/verification-notification` to resend.

Protect routes with `verified` middleware:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

## Password Confirmation

```php
Fortify::confirmPasswordView(function () {
    return view('auth.confirm-password');
});
```

POST to `/user/confirm-password` with `password`.

Use `password.confirm` middleware on protected routes.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/fortify)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
