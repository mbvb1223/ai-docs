# Laravel 12.x Eloquent Collections Documentation

## Introduction

All Eloquent methods that return more than one model result return instances of `Illuminate\Database\Eloquent\Collection`, extending Laravel's base collection class.

```php
use App\Models\User;

$users = User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

Collections support powerful map/reduce operations:

```php
$names = User::all()->reject(function (User $user) {
    return $user->active === false;
})->map(function (User $user) {
    return $user->name;
});
```

## Available Methods

| Method | Description |
|--------|-------------|
| `append($attributes)` | Append attributes to every model |
| `contains($key)` | Check if a model instance is in the collection |
| `diff($items)` | Return models not in the given collection |
| `except($keys)` | Return models without the given primary keys |
| `find($key)` | Find model(s) by primary key |
| `findOrFail($key)` | Find or throw `ModelNotFoundException` |
| `fresh($with = [])` | Retrieve fresh instances from database |
| `intersect($items)` | Return models present in both collections |
| `load($relations)` | Eager load relationships |
| `loadMissing($relations)` | Eager load relationships if not already loaded |
| `modelKeys()` | Get primary keys of all models |
| `makeVisible($attributes)` | Make hidden attributes visible |
| `makeHidden($attributes)` | Hide visible attributes |
| `only($keys)` | Return models with given primary keys |
| `partition($callback)` | Split collection into two collections |
| `toQuery()` | Convert to query builder with whereIn constraint |
| `unique()` | Return only unique models |

### Examples

```php
// Find models by key
$users = User::all();
$user = $users->find(1);

// Get primary keys
$keys = $users->modelKeys(); // [1, 2, 3, 4, 5]

// Load relationships
$users->load(['comments', 'posts']);

// Control visibility
$users = $users->makeVisible(['address', 'phone_number']);

// Convert to query and update
$users->toQuery()->update(['status' => 'Administrator']);

// Partition collection
$partition = $users->partition(fn ($user) => $user->age > 18);
```

## Custom Collections

Use a custom collection class with the `CollectedBy` attribute:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Attributes\CollectedBy;
use Illuminate\Database\Eloquent\Model;

#[CollectedBy(UserCollection::class)]
class User extends Model
{
    // ...
}
```

Or define a `newCollection()` method:

```php
public function newCollection(array $models = []): Collection
{
    return new UserCollection($models);
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-collections)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
