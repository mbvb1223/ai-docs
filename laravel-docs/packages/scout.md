# Laravel 12.x Scout Full-Text Search Documentation

## Introduction

Laravel Scout provides a driver-based solution for adding full-text search to Eloquent models.

**Supported Drivers:** Algolia, Meilisearch, Typesense, MySQL/PostgreSQL, Collection

## Installation

```bash
composer require laravel/scout
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Add the `Searchable` trait:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

## Driver Setup

### Algolia

```bash
composer require algolia/algoliasearch-client-php
```

### Meilisearch

```bash
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

```env
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
```

## Configuration

### Custom Index Name

```php
public function searchableAs(): string
{
    return 'posts_index';
}
```

### Searchable Data

```php
public function toSearchableArray(): array
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

## Indexing

### Batch Import

```bash
php artisan scout:import "App\Models\Post"
php artisan scout:flush "App\Models\Post"
```

### Adding Records

```php
$order = new Order;
$order->save(); // Automatically indexed

Order::where('price', '>', 100)->searchable();
```

### Removing Records

```php
$order->delete(); // Automatically removed

Order::where('price', '>', 100)->unsearchable();
```

### Pausing Indexing

```php
Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

### Conditionally Searchable

```php
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

## Searching

### Basic Search

```php
$orders = Order::search('Star Trek')->get();
```

### Where Clauses

```php
$orders = Order::search('Star Trek')
    ->where('user_id', 1)
    ->whereIn('status', ['open', 'paid'])
    ->get();
```

### Pagination

```php
$orders = Order::search('Star Trek')->paginate(15);
```

### Soft Deleting

```php
$orders = Order::search('Star Trek')->withTrashed()->get();
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

### Custom Eloquent Query

```php
$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/scout)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
