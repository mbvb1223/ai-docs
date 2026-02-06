# Laravel 12.x Eloquent Serialization Documentation

## Introduction

Eloquent provides methods for converting models and relationships to arrays or JSON.

## Serializing to Arrays

```php
use App\Models\User;

$user = User::with('roles')->first();
return $user->toArray();

// Attributes only (no relationships)
return $user->attributesToArray();

// Collection to array
$users = User::all();
return $users->toArray();
```

## Serializing to JSON

```php
$user = User::find(1);

return $user->toJson();
return $user->toJson(JSON_PRETTY_PRINT);

// Cast to string
return (string) User::find(1);
```

Returning from routes automatically serializes to JSON:

```php
Route::get('/users', function () {
    return User::all();
});
```

## Hiding Attributes From JSON

### Using `$hidden`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $hidden = ['password'];
}
```

### Using `$visible`

```php
class User extends Model
{
    protected $visible = ['first_name', 'last_name'];
}
```

### Temporarily Modifying Visibility

```php
// Make visible
return $user->makeVisible('attribute')->toArray();
return $user->mergeVisible(['name', 'email'])->toArray();

// Make hidden
return $user->makeHidden('attribute')->toArray();
return $user->mergeHidden(['name', 'email'])->toArray();

// Override all
return $user->setVisible(['id', 'name'])->toArray();
return $user->setHidden(['email', 'password'])->toArray();
```

## Appending Values to JSON

### Define an Accessor

```php
protected function isAdmin(): Attribute
{
    return new Attribute(
        get: fn () => 'yes',
    );
}
```

### Add to `$appends`

```php
class User extends Model
{
    protected $appends = ['is_admin'];
}
```

### Appending at Run Time

```php
return $user->append('is_admin')->toArray();
return $user->mergeAppends(['is_admin', 'status'])->toArray();
return $user->setAppends(['is_admin'])->toArray();
return $user->withoutAppends()->toArray();
```

## Date Serialization

### Default Date Format

```php
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

### Per-Attribute Format

```php
protected function casts(): array
{
    return [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-serialization)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
