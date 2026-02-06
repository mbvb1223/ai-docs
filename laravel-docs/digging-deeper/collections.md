# Laravel 12.x Collections Documentation

## Introduction

The `Illuminate\Support\Collection` class provides a fluent, convenient wrapper for working with arrays of data. Collections are immutable, meaning every `Collection` method returns an entirely new `Collection` instance.

## Creating Collections

```php
// Using the collect helper
$collection = collect([1, 2, 3]);

// Using the make() method
$collection = Collection::make([1, 2, 3]);

// From JSON
$collection = Collection::fromJson($json);
```

## Basic Example

```php
$collection = collect(['Taylor', 'Abigail', null])
    ->map(function (?string $name) {
        return strtoupper($name);
    })
    ->reject(function (string $name) {
        return empty($name);
    });
```

## Extending Collections

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);
$upper = $collection->toUpper();
// ['FIRST', 'SECOND']
```

## Available Methods

### Data Retrieval

- **`all()`** - Returns underlying array
- **`get(key, default)`** - Get item by key
- **`first(callback)`** - Get first item
- **`last(callback)`** - Get last item
- **`nth(n, offset)`** - Get every n-th element
- **`random(count)`** - Get random item(s)
- **`count()`** - Count items
- **`isEmpty()` / `isNotEmpty()`** - Check if empty

### Transformation

- **`map(callback)`** - Transform each item
- **`filter(callback)`** - Filter items by condition
- **`reject(callback)`** - Filter out items
- **`flatten(depth)`** - Flatten multi-dimensional collection
- **`flatMap(callback)`** - Map and flatten in one step
- **`transform(callback)`** - Transform in place
- **`reverse()`** - Reverse items
- **`shuffle()`** - Randomize order
- **`sort()` / `sortBy(key)`** - Sort items

### Grouping & Organization

- **`chunk(size)`** - Break into smaller collections
- **`groupBy(key/callback)`** - Group items
- **`keyBy(key/callback)`** - Key collection by value
- **`partition(callback)`** - Separate into two groups

### Aggregation

- **`sum(key)`** - Sum values
- **`avg(key)` / `average(key)`** - Average value
- **`min(key)` / `max(key)`** - Min/max values
- **`median(key)`** - Median value
- **`mode(key)`** - Mode value
- **`reduce(callback)`** - Reduce to single value

### Searching

- **`contains(value/callback)`** - Check if contains value
- **`doesntContain(value)`** - Inverse of contains
- **`has(key/keys)`** - Check if key exists
- **`search(value)`** - Find key of value
- **`where(key, operator, value)`** - Filter by condition

### Set Operations

- **`diff(items)`** - Values not in given collection
- **`intersect(items)`** - Values in both collections
- **`union(items)`** - Union of collections
- **`unique(key)`** - Remove duplicates
- **`merge(items)`** - Merge collections

## Common Method Examples

### Map

```php
$collection = collect([1, 2, 3, 4, 5]);
$multiplied = $collection->map(function (int $item) {
    return $item * 2;
});
// [2, 4, 6, 8, 10]
```

### Filter

```php
$collection = collect([1, 2, 3, 4]);
$filtered = $collection->filter(function (int $value) {
    return $value > 2;
});
// [3, 4]
```

### Reduce

```php
$collection = collect([1, 2, 3]);
$result = $collection->reduce(function ($carry, $item) {
    return $carry + $item;
});
// 6
```

### GroupBy

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');
// account-x10 => [Chair, Bookcase], account-x11 => [Desk]
```

### Pluck

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');
// ['Desk', 'Chair']

$plucked = $collection->pluck('name', 'product_id');
// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

### Where

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->where('price', '>', 100);
// Desk, Bookcase
```

### SortBy

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
]);

$sorted = $collection->sortBy('price');
// Chair, Desk
```

### Each

```php
$collection->each(function (int $item, int $key) {
    // Process each item
});
```

### Contains

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk'); // true
$collection->contains('New York'); // false
```

### Keys / Values

```php
$collection = collect(['a' => 1, 'b' => 2, 'c' => 3]);

$collection->keys(); // ['a', 'b', 'c']
$collection->values(); // [1, 2, 3]
```

### Flip

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$flipped = $collection->flip();
// ['taylor' => 'name', 'laravel' => 'framework']
```

## Lazy Collections

### Introduction

Lazy collections defer processing until the collection is actually enumerated, reducing memory usage for large datasets.

### Creating Lazy Collections

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

// From existing collection
$lazyCollection = collect([1, 2, 3, 4])->lazy();
```

### Benefits

```php
$count = $hugeCollection
    ->lazy()
    ->where('country', 'FR')
    ->where('balance', '>', '100')
    ->count();
```

### Converting to Collection

```php
$collection = $lazyCollection->collect();
```

## Higher Order Messages

Collections provide "higher order messages" as shortcuts for common actions:

```php
$users->each->markAsVip();
$users->filter->isAdmin();
$users->sum->votes;
```

## Tips

1. **Chain methods** for fluent operations
2. **Use `lazy()`** for processing large datasets
3. **Use `map()`** for transformations, **`filter()`** for conditions
4. **Use `pluck()`** to extract specific fields
5. **Use `groupBy()`** to organize data
6. **Use `reduce()`** for aggregations
7. Collections are **immutable** - methods return new instances
8. **Macros** let you extend functionality

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/collections)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
