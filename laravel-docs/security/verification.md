# Laravel 12.x Email Verification Documentation

## Introduction

Laravel provides built-in services for sending and verifying email verification requests.

## Model Preparation

Implement the `MustVerifyEmail` contract on your User model:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;
}
```

Dispatch the `Registered` event for manual registration:

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

## Database Preparation

Your `users` table must contain an `email_verified_at` column.

## Routing

### Email Verification Notice Route

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

### Email Verification Handler Route

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

### Resending Verification Email

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

## Protecting Routes

```php
Route::get('/profile', function () {
    // Only verified users may access this route...
})->middleware(['auth', 'verified']);
```

## Customization

### Verification Email Customization

```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

public function boot(): void
{
    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('Verify Email Address')
            ->line('Click the button below to verify your email address.')
            ->action('Verify Email Address', $url);
    });
}
```

## Events

Laravel dispatches the `Illuminate\Auth\Events\Verified` event during email verification.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/verification)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
