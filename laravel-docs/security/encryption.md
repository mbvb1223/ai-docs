# Laravel 12.x Encryption Documentation

## Introduction

Laravel's encryption services provide a simple interface for encrypting and decrypting text via OpenSSL using AES-256 and AES-128 encryption. All encrypted values are signed using a message authentication code (MAC) to prevent tampering.

## Configuration

Set the `key` configuration option in `config/app.php`, driven by the `APP_KEY` environment variable.

**Generate the encryption key:**
```bash
php artisan key:generate
```

### Gracefully Rotating Encryption Keys

When you change your encryption key, authenticated user sessions are logged out. Use `APP_PREVIOUS_KEYS` to mitigate:

```env
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

Laravel will:
- Always use the current key when encrypting
- Try the current key first when decrypting
- Fall back to previous keys if needed

## Using the Encrypter

### Encrypting a Value

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

### Decrypting a Value

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // Handle decryption error
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/encryption)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
