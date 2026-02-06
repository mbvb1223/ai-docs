# Laravel 12.x Eloquent Factories Documentation

## Introduction

Model factories define default attributes for Eloquent models for testing and seeding.

## Generating Factories

```bash
php artisan make:factory PostFactory
```

## Defining Factories

### Basic Structure

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => Hash::make('password'),
        ];
    }
}
```

### Factory States

```php
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    });
}
```

### Factory Callbacks

```php
public function configure(): static
{
    return $this->afterMaking(function (User $user) {
        // ...
    })->afterCreating(function (User $user) {
        // ...
    });
}
```

## Creating Models

### Instantiating Without Persisting

```php
$user = User::factory()->make();
$users = User::factory()->count(3)->make();
```

### Persisting to Database

```php
$user = User::factory()->create();
$users = User::factory()->count(3)->create();
```

### Applying States

```php
$users = User::factory()->count(5)->suspended()->make();
```

### Overriding Attributes

```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```

### Sequences

```php
$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

## Factory Relationships

### Has Many

```php
$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();

// Magic method
$user = User::factory()
    ->hasPosts(3)
    ->create();
```

### Belongs To

```php
$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();

// Magic method
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

### Many to Many

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

### Defining Relationships in Factories

```php
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->title(),
    ];
}
```

### Recycling Models

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-factories)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
