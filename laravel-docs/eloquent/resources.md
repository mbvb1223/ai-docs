# Laravel 12.x Eloquent API Resources Documentation

## Introduction

API Resources provide a transformation layer between Eloquent models and JSON responses.

## Generating Resources

```bash
php artisan make:resource UserResource
php artisan make:resource UserCollection
```

## Basic Resource Structure

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

## Using Resources

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});

// Or using toResource()
return User::findOrFail($id)->toResource();
```

### Resource Collections

```php
return UserResource::collection(User::all());

// Or
return User::all()->toResourceCollection();
```

## Including Relationships

```php
use App\Http\Resources\PostResource;

public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'posts' => PostResource::collection($this->posts),
    ];
}
```

## Data Wrapping

### Disable Wrapping

```php
use Illuminate\Http\Resources\Json\JsonResource;

public function boot(): void
{
    JsonResource::withoutWrapping();
}
```

## Pagination

```php
return new UserCollection(User::paginate());
```

## Conditional Attributes

### Using `when()`

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
    ];
}
```

### Merging Conditional Attributes

```php
$this->mergeWhen($request->user()->isAdmin(), [
    'first-secret' => 'value',
    'second-secret' => 'value',
]),
```

## Conditional Relationships

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'posts_count' => $this->whenCounted('posts'),
    ];
}
```

### Aggregates

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
```

### Conditional Pivot Information

```php
'expires_at' => $this->whenPivotLoaded('role_user', function () {
    return $this->pivot->expires_at;
}),
```

## Adding Meta Data

### Top-Level Meta Data

```php
public function with(Request $request): array
{
    return [
        'meta' => [
            'key' => 'value',
        ],
    ];
}
```

### Adding Meta When Constructing

```php
return User::all()
    ->toResourceCollection()
    ->additional(['meta' => ['key' => 'value']]);
```

## Customizing HTTP Response

```php
return User::find(1)
    ->toResource()
    ->response()
    ->header('X-Value', 'True');
```

### Using `withResponse()`

```php
public function withResponse(Request $request, JsonResponse $response): void
{
    $response->header('X-Value', 'True');
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-resources)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
