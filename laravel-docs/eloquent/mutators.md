# Laravel 12.x Eloquent Mutators & Casting Documentation

## Introduction

Accessors, mutators, and attribute casting transform Eloquent attribute values when retrieving or setting them.

## Accessors and Mutators

### Defining an Accessor

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```

### Building Value Objects

```php
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

### Defining a Mutator

```php
protected function firstName(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => ucfirst($value),
        set: fn (string $value) => strtolower($value),
    );
}
```

### Mutating Multiple Attributes

```php
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

## Attribute Casting

### Supported Cast Types

- `array`, `collection`, `object`
- `boolean`, `integer`, `float`, `double`, `string`
- `date`, `datetime`, `immutable_date`, `immutable_datetime`, `timestamp`
- `decimal:<precision>`
- `encrypted`, `encrypted:array`, `encrypted:collection`
- `hashed`
- `json:unicode`

### Basic Example

```php
protected function casts(): array
{
    return [
        'is_admin' => 'boolean',
        'options' => 'array',
        'created_at' => 'datetime:Y-m-d',
    ];
}
```

### Array and JSON Casting

```php
protected function casts(): array
{
    return [
        'options' => 'array',
    ];
}

// Usage
$user->options['key'] = 'value';
$user->save();

// Update single JSON field
$user->update(['options->key' => 'value']);
```

### Collection Casting

```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

protected function casts(): array
{
    return [
        'options' => AsCollection::class,
    ];
}
```

### Enum Casting

```php
use App\Enums\ServerStatus;

protected function casts(): array
{
    return [
        'status' => ServerStatus::class,
    ];
}
```

### Encrypted Casting

```php
protected function casts(): array
{
    return [
        'password' => 'encrypted',
        'data' => 'encrypted:array',
    ];
}
```

### Query Time Casting

```php
$users = User::select(['users.*', 'last_posted_at' => ...])
    ->withCasts(['last_posted_at' => 'datetime'])
    ->get();
```

## Custom Casts

### Creating a Custom Cast

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class AsJson implements CastsAttributes
{
    public function get(Model $model, string $key, mixed $value, array $attributes): array
    {
        return json_decode($value, true);
    }

    public function set(Model $model, string $key, mixed $value, array $attributes): string
    {
        return json_encode($value);
    }
}
```

### Using Custom Cast

```php
protected function casts(): array
{
    return [
        'options' => AsJson::class,
    ];
}
```

### Value Object Casting

```php
<?php

namespace App\Casts;

use App\ValueObjects\Address;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class AsAddress implements CastsAttributes
{
    public function get(Model $model, string $key, mixed $value, array $attributes): Address
    {
        return new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two']
        );
    }

    public function set(Model $model, string $key, mixed $value, array $attributes): array
    {
        return [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ];
    }
}
```

### Cast Parameters

```php
protected function casts(): array
{
    return [
        'secret' => AsHash::class.':sha256',
    ];
}
```

### Castables

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;

class Address implements Castable
{
    public static function castUsing(array $arguments): string
    {
        return AsAddress::class;
    }
}

// Usage
protected function casts(): array
{
    return [
        'address' => Address::class,
    ];
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-mutators)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
