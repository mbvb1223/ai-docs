# Laravel 12.x Database: Getting Started

## Introduction

Laravel provides first-party support for five databases:
- MariaDB 10.3+
- MySQL 5.7+
- PostgreSQL 10.0+
- SQLite 3.26.0+
- SQL Server 2017+

Additionally, MongoDB is supported via the `mongodb/laravel-mongodb` package maintained by MongoDB.

## Configuration

Database configuration is located in `config/database.php`. The file defines all database connections and specifies which connection is used by default. Most configuration options are driven by environment variables.

### SQLite Configuration

SQLite databases are single files on your filesystem. Create one with:
```bash
touch database/database.sqlite
```

Configure the environment variable:
```
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

Enable/disable foreign key constraints:
```
DB_FOREIGN_KEYS=false
```

### Configuration Using URLs

Database providers like AWS and Heroku provide single database URLs containing all connection information:

```
mysql://root:[email protected]/forge?charset=UTF-8
```

Standard schema convention:
```
driver://username:password@host:port/database?options
```

Laravel will extract connection and credential information from the `DB_URL` environment variable if present.

## Read and Write Connections

Configure separate read and write connections:

```php
'mysql' => [
    'driver' => 'mysql',

    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,

    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
],
```

### The `sticky` Option

When enabled, the `sticky` option allows immediate reading of records written during the current request cycle. If a write operation occurs, subsequent read operations use the write connection, ensuring written data can be immediately read back.

## Running SQL Queries

### Running a Select Query

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

The `select` method returns an array of `stdClass` objects:

```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

### Selecting Scalar Values

Retrieve single scalar values directly:

```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

### Selecting Multiple Result Sets

For stored procedures returning multiple result sets:

```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

### Using Named Bindings

Instead of `?` placeholders:

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

### Running an Insert Statement

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

### Running an Update Statement

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

### Running a Delete Statement

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

### Running a General Statement

```php
DB::statement('drop table users');
```

### Running an Unprepared Statement

```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```

**Warning**: Unprepared statements do not bind parameters and are vulnerable to SQL injection. Never allow user-controlled values.

### Implicit Commits

Avoid statements causing implicit commits within transactions (e.g., creating database tables):

```php
DB::unprepared('create table a (col varchar(1) null)');
```

## Using Multiple Database Connections

Access specific connections via the `connection` method:

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```

Access the raw PDO instance:

```php
$pdo = DB::connection()->getPdo();
```

## Listening for Query Events

Register a closure to handle each SQL query executed:

```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        DB::listen(function (QueryExecuted $query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
            // $query->toRawSql();
        });
    }
}
```

## Monitoring Cumulative Query Time

Invoke a callback when query time exceeds a threshold:

```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Notify development team...
        });
    }
}
```

## Database Transactions

Execute operations within a transaction with automatic rollback/commit:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

### Handling Deadlocks

Retry transactions on deadlock:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, attempts: 5);
```

### Manually Using Transactions

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();

// ... perform operations ...

DB::rollBack();  // or
DB::commit();
```

## Connecting to the Database CLI

Connect to your database using the Artisan command:

```bash
php artisan db
```

Specify a database connection:

```bash
php artisan db mysql
```

## Inspecting Your Databases

### Database Overview

```bash
php artisan db:show
```

Specify a connection:

```bash
php artisan db:show --database=pgsql
```

Include row counts and view details:

```bash
php artisan db:show --counts --views
```

### Using Schema Methods

```php
use Illuminate\Support\Facades\Schema;

$tables = Schema::getTables();
$views = Schema::getViews();
$columns = Schema::getColumns('users');
$indexes = Schema::getIndexes('users');
$foreignKeys = Schema::getForeignKeys('users');
```

Inspect non-default connections:

```php
$columns = Schema::connection('sqlite')->getColumns('users');
```

### Table Overview

Get detailed overview of a specific table:

```bash
php artisan db:table users
```

Shows columns, types, attributes, keys, and indexes.

## Monitoring Your Databases

Use the `db:monitor` command to track open database connections:

```bash
php artisan db:monitor --databases=mysql,pgsql --max=100
```

Schedule this command to run every minute. When the connection count exceeds the threshold, a `DatabaseBusy` event is dispatched.

Listen for the event in your `AppServiceProvider`:

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', '[email protected]')
            ->notify(new DatabaseApproachingMaxConnections(
                $event->connectionName,
                $event->connections
            ));
    });
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/database)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
