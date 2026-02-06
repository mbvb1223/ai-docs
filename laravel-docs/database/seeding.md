# Laravel 12.x Database Seeding Documentation

## Introduction

Laravel includes the ability to seed your database with data using seed classes stored in `database/seeders`. Mass assignment protection is automatically disabled during seeding.

## Writing Seeders

### Creating a Seeder

```bash
php artisan make:seeder UserSeeder
```

### Basic Example

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

### Using Model Factories

```php
use App\Models\User;

public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

### Calling Additional Seeders

```php
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

### Muting Model Events

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

## Running Seeders

### Execute Seeders

```bash
php artisan db:seed
```

### Run Specific Seeder

```bash
php artisan db:seed --class=UserSeeder
```

### With Migrations

```bash
php artisan migrate:fresh --seed
php artisan migrate:fresh --seed --seeder=UserSeeder
```

### Force in Production

```bash
php artisan db:seed --force
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/seeding)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
