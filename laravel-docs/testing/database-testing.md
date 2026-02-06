# Laravel 12.x Database Testing Documentation

## Introduction

Laravel provides helpful tools and assertions for testing database-driven applications.

## Resetting the Database After Each Test

Use the `RefreshDatabase` trait:

```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('basic example', function () {
    $response = $this->get('/');
    // ...
});
```

## Model Factories

```php
use App\Models\User;

test('models can be instantiated', function () {
    $user = User::factory()->create();
    // ...
});
```

## Running Seeders

```php
<?php

use Database\Seeders\OrderStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('orders can be created', function () {
    // Run the DatabaseSeeder
    $this->seed();

    // Run a specific seeder
    $this->seed(OrderStatusSeeder::class);

    // Run multiple seeders
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
    ]);
});
```

### Auto-Seeding

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected $seed = true;

    // Optional: specify seeder
    protected $seeder = OrderStatusSeeder::class;
}
```

## Available Assertions

### assertDatabaseCount

```php
$this->assertDatabaseCount('users', 5);
```

### assertDatabaseEmpty

```php
$this->assertDatabaseEmpty('users');
```

### assertDatabaseHas

```php
$this->assertDatabaseHas('users', [
    'email' => '[email protected]',
]);
```

### assertDatabaseMissing

```php
$this->assertDatabaseMissing('users', [
    'email' => '[email protected]',
]);
```

### assertSoftDeleted

```php
$this->assertSoftDeleted($user);
```

### assertNotSoftDeleted

```php
$this->assertNotSoftDeleted($user);
```

### assertModelExists

```php
$user = User::factory()->create();
$this->assertModelExists($user);
```

### assertModelMissing

```php
$user = User::factory()->create();
$user->delete();
$this->assertModelMissing($user);
```

### expectsDatabaseQueryCount

```php
$this->expectsDatabaseQueryCount(5);
// Test...
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/database-testing)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
