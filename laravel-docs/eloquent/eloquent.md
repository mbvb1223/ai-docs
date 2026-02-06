# Eloquent: Getting Started - Laravel 12.x

## Introduction

Laravel includes **Eloquent**, an object-relational mapper (ORM) that makes it enjoyable to interact with your database. When using Eloquent, each database table has a corresponding "Model" that is used to interact with that table. In addition to retrieving records from the database table, Eloquent models allow you to insert, update, and delete records from the table as well.

Before getting started, be sure to configure a database connection in your application's `config/database.php` configuration file.

## Generating Model Classes

To get started, let's create an Eloquent model. Models typically live in the `app\Models` directory and extend the `Illuminate\Database\Eloquent\Model` class.

```bash
php artisan make:model Flight
```

If you would like to generate a database migration when you generate the model:

```bash
php artisan make:model Flight --migration
php artisan make:model Flight -m
```

You may generate various other types of classes when generating a model:

```bash
# Generate a model and a FlightFactory class...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Generate a model and a FlightSeeder class...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Generate a model and a FlightController class...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Generate a model, FlightController resource class, and form request classes...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Generate a model and a FlightPolicy class...
php artisan make:model Flight --policy

# Generate a model and a migration, factory, seeder, and controller...
php artisan make:model Flight -mfsc

# Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
php artisan make:model Flight --all
php artisan make:model Flight -a

# Generate a pivot model...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

### Inspecting Models

```bash
php artisan model:show Flight
```

## Eloquent Model Conventions

### Basic Model Class

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // ...
}
```

### Table Names

By convention, the "snake case", plural name of the class will be used as the table name. Override with a `table` property:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $table = 'my_flights';
}
```

### Primary Keys

Eloquent assumes each table has a primary key column named `id`. Customize with `$primaryKey`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $primaryKey = 'flight_id';
}
```

If your primary key is not an integer:

```php
<?php

class Flight extends Model
{
    public $incrementing = false;
    protected $keyType = 'string';
}
```

### UUID and ULID Keys

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;
}

$article = Article::create(['title' => 'Traveling to Europe']);
$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"
```

For ULIDs:

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;
}

$article = Article::create(['title' => 'Traveling to Asia']);
$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

### Timestamps

By default, Eloquent expects `created_at` and `updated_at` columns. Disable with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    public $timestamps = false;
}
```

Customize the format:

```php
protected $dateFormat = 'U';
```

Customize column names:

```php
public const CREATED_AT = 'creation_date';
public const UPDATED_AT = 'updated_date';
```

### Database Connections

```php
protected $connection = 'mysql';
```

### Default Attribute Values

```php
protected $attributes = [
    'options' => '[]',
    'delayed' => false,
];
```

### Configuring Eloquent Strictness

```php
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
    Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
}
```

## Retrieving Models

### Retrieving All Records

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

### Building Queries

```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();
```

### Refreshing Models

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();

$flight->refresh();
```

## Collections

Eloquent methods like `all` and `get` return `Illuminate\Database\Eloquent\Collection`:

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

## Chunking Results

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

### Lazy Collections

```php
foreach (Flight::lazy() as $flight) {
    // ...
}
```

### Cursors

```php
foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

## Retrieving Single Models

```php
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```

### Not Found Exceptions

```php
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

## Retrieving or Creating Models

```php
// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or create it with additional attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Retrieve flight by name or instantiate (not persisted)...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);
```

## Inserting and Updating Models

### Inserts

```php
<?php

namespace App\Http\Controllers;

use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    public function store(Request $request): RedirectResponse
    {
        $flight = new Flight;
        $flight->name = $request->name;
        $flight->save();

        return redirect('/flights');
    }
}
```

Or use `create`:

```php
$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

### Updates

```php
$flight = Flight::find(1);
$flight->name = 'Paris to London';
$flight->save();
```

### Mass Updates

```php
Flight::where('active', 1)
    ->where('destination', 'San Diego')
    ->update(['delayed' => 1]);
```

### Upserts

```php
Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);

Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

## Mass Assignment

### Fillable Attributes

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $fillable = ['name'];
}
```

### Guarded Attributes

```php
protected $guarded = [];
```

## Deleting Models

```php
use App\Models\Flight;

$flight = Flight::find(1);
$flight->delete();

// Delete by primary key
Flight::destroy(1);
Flight::destroy([1, 2, 3]);

// Delete using queries
$deleted = Flight::where('active', 0)->delete();
```

## Soft Deleting

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

### Querying Soft Deleted Models

```php
// Include soft deleted
$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();

// Only soft deleted
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

### Restoring Soft Deleted Models

```php
$flight->restore();

Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

### Permanently Deleting

```php
$flight->forceDelete();
```

## Query Scopes

### Global Scopes

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->minus(years: 2000));
    }
}
```

Apply with attribute:

```php
use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Attributes\ScopedBy;

#[ScopedBy([AncientScope::class])]
class User extends Model
{
    //
}
```

### Local Scopes

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    #[Scope]
    protected function active(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

Usage:

```php
$users = User::popular()->active()->orderBy('created_at')->get();
```

### Dynamic Scopes

```php
#[Scope]
protected function ofType(Builder $query, string $type): void
{
    $query->where('type', $type);
}

$users = User::ofType('admin')->get();
```

## Events

Eloquent models dispatch events:
- `retrieved`
- `creating` / `created`
- `updating` / `updated`
- `saving` / `saved`
- `deleting` / `deleted`
- `trashed` / `restored` / `forceDeleted`
- `replicating`

### Using Closures

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected static function booted(): void
    {
        static::created(function (User $user) {
            // ...
        });
    }
}
```

### Observers

```bash
php artisan make:observer UserObserver --model=User
```

```php
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    public function created(User $user): void
    {
        // ...
    }

    public function updated(User $user): void
    {
        // ...
    }

    public function deleted(User $user): void
    {
        // ...
    }
}
```

Register with attribute:

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
