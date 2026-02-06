# Laravel 12.x Hashing Documentation

## Introduction

The Laravel `Hash` facade provides secure Bcrypt and Argon2 hashing for storing user passwords. Bcrypt is the default algorithm.

## Configuration

By default, Laravel uses the `bcrypt` hashing driver. Other supported drivers include `argon` and `argon2id`.

Configure via `HASH_DRIVER` environment variable or publish configuration:

```bash
php artisan config:publish hashing
```

## Basic Usage

### Hashing Passwords

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    public function update(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

### Adjusting Bcrypt Work Factor

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

### Adjusting Argon2 Work Factor

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

### Verifying Passwords

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}
```

### Determining if Rehashing is Needed

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

## Hash Algorithm Verification

Laravel verifies hashes were generated using the configured algorithm. Disable verification for multiple algorithms:

```
HASH_VERIFY=false
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/hashing)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
