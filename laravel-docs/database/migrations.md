# Laravel 12.x Database Migrations

## Introduction

Migrations are version control for your database, allowing teams to define and share the application's database schema definition. The Laravel `Schema` facade provides database-agnostic support for creating and manipulating tables across all supported database systems.

## Generating Migrations

Generate a migration using the `make:migration` Artisan command:

```bash
php artisan make:migration create_flights_table
```

The new migration will be placed in the `database/migrations` directory with a timestamp filename that determines execution order.

### Squashing Migrations

```bash
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

## Migration Structure

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

## Running Migrations

```bash
# Execute all outstanding migrations
php artisan migrate

# View migration status
php artisan migrate:status

# Preview SQL without executing
php artisan migrate --pretend

# Force migrations in production
php artisan migrate --force
```

### Rolling Back Migrations

```bash
# Roll back the latest batch
php artisan migrate:rollback

# Roll back specific number of migrations
php artisan migrate:rollback --step=5

# Roll back all migrations
php artisan migrate:reset

# Roll back and re-migrate all migrations
php artisan migrate:refresh

# Drop all tables and migrate
php artisan migrate:fresh
```

## Tables

### Creating Tables

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

### Checking Table/Column Existence

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table has an "email" column...
}
```

### Updating Tables

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### Renaming / Dropping Tables

```php
Schema::rename($from, $to);
Schema::drop('users');
Schema::dropIfExists('users');
```

## Columns

### Available Column Types

#### Primary Key & ID Types
- `$table->id()` - Auto-incrementing UNSIGNED BIGINT
- `$table->uuid()` - UUID equivalent
- `$table->ulid()` - ULID equivalent

#### String Types
- `$table->string('name', 100)`
- `$table->char('code', 10)`
- `$table->text('description')`
- `$table->mediumText('content')`
- `$table->longText('content')`

#### Numeric Types
- `$table->integer('votes')`
- `$table->bigInteger('votes')`
- `$table->tinyInteger('votes')`
- `$table->smallInteger('votes')`
- `$table->mediumInteger('votes')`
- `$table->unsignedInteger('votes')`
- `$table->unsignedBigInteger('votes')`
- `$table->float('amount', precision: 53)`
- `$table->double('amount')`
- `$table->decimal('amount', total: 8, places: 2)`

#### Date & Time Types
- `$table->date('created_at')`
- `$table->dateTime('created_at', precision: 0)`
- `$table->time('sunrise', precision: 0)`
- `$table->timestamp('added_at', precision: 0)`
- `$table->timestamps(precision: 0)`
- `$table->softDeletes(precision: 0)`
- `$table->year('birth_year')`

#### Other Types
- `$table->boolean('confirmed')`
- `$table->binary('photo')`
- `$table->json('options')`
- `$table->jsonb('options')`
- `$table->enum('level', ['easy', 'hard'])`
- `$table->ipAddress('visitor')`
- `$table->macAddress('device')`
- `$table->rememberToken()`

### Column Modifiers

```php
$table->string('email')->nullable();
$table->string('email')->default('default@example.com');
$table->integer('votes')->unsigned();
$table->string('email')->unique();
$table->text('bio')->comment('User biography');
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrentOnUpdate();
```

### Modifying Columns

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

### Renaming Columns

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

### Dropping Columns

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

## Indexes

### Creating Indexes

```php
$table->string('email')->unique();

$table->unique('email');
$table->index(['account_id', 'created_at']);
$table->unique('email', 'unique_email'); // Custom name
```

### Available Index Types

```php
$table->primary('id');
$table->primary(['id', 'parent_id']); // Composite keys
$table->unique('email');
$table->index('state');
$table->fullText('body');
$table->spatialIndex('location');
```

### Dropping Indexes

```php
$table->dropPrimary('users_id_primary');
$table->dropUnique('users_email_unique');
$table->dropIndex('geo_state_index');
$table->dropFullText('posts_body_fulltext');

// Drop by columns
$table->dropIndex(['state']); // Drops 'geo_state_index'
```

## Foreign Key Constraints

### Creating Foreign Keys

```php
Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('users');
});
```

### Simplified Foreign Keys

```php
$table->foreignId('user_id')->constrained();

// With custom table and index name
$table->foreignId('user_id')->constrained(
    table: 'users',
    indexName: 'posts_user_id'
);
```

### Cascade Actions

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');

// Or use helper methods
$table->foreignId('user_id')
    ->constrained()
    ->cascadeOnUpdate()
    ->cascadeOnDelete();
```

### Available Actions

| Method | Description |
|--------|-------------|
| `cascadeOnUpdate()` | Updates cascade |
| `restrictOnUpdate()` | Restrict updates |
| `nullOnUpdate()` | Set to NULL on update |
| `cascadeOnDelete()` | Deletes cascade |
| `restrictOnDelete()` | Restrict deletes |
| `nullOnDelete()` | Set to NULL on delete |

### Dropping Foreign Keys

```php
$table->dropForeign('posts_user_id_foreign');
$table->dropForeign(['user_id']);
```

### Toggling Foreign Key Constraints

```php
Schema::enableForeignKeyConstraints();
Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // Constraints disabled within this closure...
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/migrations)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
