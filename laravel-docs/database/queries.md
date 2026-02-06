# Laravel 12.x Query Builder Documentation

## Introduction

Laravel's Query Builder provides a fluent, convenient interface for creating and running database queries. It uses PDO parameter binding to protect against SQL injection attacks and works with all supported database systems.

## Running Database Queries

### Retrieving Results

**Get all rows:**
```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

**Get a single row:**
```php
$user = DB::table('users')->where('name', 'John')->first();
return $user->email;

// Or throw exception if not found
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```

**Get a single value:**
```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

**Find by ID:**
```php
$user = DB::table('users')->find(3);
```

**Get column values:**
```php
$titles = DB::table('users')->pluck('title');
$titles = DB::table('users')->pluck('title', 'name'); // with key
```

### Chunking Results

Process large datasets in chunks:
```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // Process each user
    }
});

// Stop processing by returning false
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    return false; // Stop chunking
});
```

Use `chunkById()` when updating records:
```php
DB::table('users')->where('active', false)
    ->chunkById(100, function (Collection $users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

### Streaming Results Lazily

```php
DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // Process each user
});

// When updating, use lazyById
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

### Aggregates

```php
$count = DB::table('users')->count();
$price = DB::table('orders')->max('price');
$price = DB::table('orders')->min('price');
$avg = DB::table('orders')->avg('price');
$sum = DB::table('orders')->sum('price');

// With conditions
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');

// Check existence
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

## Select Statements

```php
// Select specific columns
$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();

// Distinct results
$users = DB::table('users')->distinct()->get();

// Add columns to existing select
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();
```

## Raw Expressions

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

### Raw Methods

**selectRaw:**
```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

**whereRaw / orWhereRaw:**
```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

**havingRaw / orHavingRaw:**
```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

**orderByRaw:**
```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

**groupByRaw:**
```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

## Joins

### Inner Join
```php
$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

### Left/Right Join
```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

### Cross Join
```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

### Advanced Join Clauses
```php
use Illuminate\Database\Query\JoinClause;

DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->orOn(/* ... */);
    })
    ->get();

DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

### Subquery Joins
```php
$latestPosts = DB::table('posts')
    ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = DB::table('users')
    ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

### Lateral Joins (PostgreSQL, MySQL 8.0.14+, SQL Server)
```php
$latestPosts = DB::table('posts')
    ->select('id as post_id', 'title as post_title', 'created_at as post_created_at')
    ->whereColumn('user_id', 'users.id')
    ->orderBy('created_at', 'desc')
    ->limit(3);

$users = DB::table('users')
    ->joinLateral($latestPosts, 'latest_posts')
    ->get();
```

## Unions

```php
$first = DB::table('users')->whereNull('first_name');

$users = DB::table('users')
    ->whereNull('last_name')
    ->union($first)
    ->get();

// Without removing duplicates
$users = DB::table('users')
    ->whereNull('last_name')
    ->unionAll($first)
    ->get();
```

## Where Clauses

### Basic Where
```php
$users = DB::table('users')
    ->where('votes', '=', 100)
    ->where('age', '>', 35)
    ->get();

// Shorthand (= is assumed)
$users = DB::table('users')->where('votes', 100)->get();

// Array of conditions
$users = DB::table('users')->where([
    'first_name' => 'Jane',
    'last_name' => 'Doe',
])->get();

// Array of conditions with operators
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

### Or Where
```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere('name', 'John')
    ->get();

// Or with grouping
use Illuminate\Database\Query\Builder;

$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere(function (Builder $query) {
        $query->where('name', 'Abigail')
            ->where('votes', '>', 50);
    })
    ->get();
```

### Where Not
```php
$products = DB::table('products')
    ->whereNot(function (Builder $query) {
        $query->where('clearance', true)
            ->orWhere('price', '<', 10);
    })
    ->get();
```

### Where Any / All / None
```php
// Any column matches
$users = DB::table('users')
    ->where('active', true)
    ->whereAny([
        'name',
        'email',
        'phone',
    ], 'like', 'Example%')
    ->get();

// All columns match
$posts = DB::table('posts')
    ->where('published', true)
    ->whereAll([
        'title',
        'content',
    ], 'like', '%Laravel%')
    ->get();

// None match
$albums = DB::table('albums')
    ->where('published', true)
    ->whereNone([
        'title',
        'lyrics',
        'tags',
    ], 'like', '%explicit%')
    ->get();
```

### JSON Where Clauses
```php
$users = DB::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();

$users = DB::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();

$users = DB::table('users')
    ->whereJsonContainsKey('preferences->dietary_requirements')
    ->get();

$users = DB::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

### Additional Where Clauses

**whereLike / whereNotLike:**
```php
$users = DB::table('users')
    ->whereLike('name', '%John%')
    ->get();

// Case-sensitive
$users = DB::table('users')
    ->whereLike('name', '%John%', caseSensitive: true)
    ->get();
```

**whereIn / whereNotIn:**
```php
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();

$users = DB::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

**whereBetween / whereNotBetween:**
```php
$users = DB::table('users')
    ->whereBetween('votes', [1, 100])
    ->get();
```

**whereNull / whereNotNull:**
```php
$users = DB::table('users')
    ->whereNull('updated_at')
    ->get();
```

**Date Comparisons:**
```php
$users = DB::table('users')
    ->whereDate('created_at', '2016-12-31')
    ->get();

$users = DB::table('users')
    ->whereMonth('created_at', '12')
    ->get();

$users = DB::table('users')
    ->whereYear('created_at', '2016')
    ->get();

$users = DB::table('users')
    ->whereTime('created_at', '=', '11:20:45')
    ->get();
```

**Past/Future Date Comparisons:**
```php
$invoices = DB::table('invoices')
    ->wherePast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereFuture('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereToday('due_at')
    ->get();
```

**whereColumn:**
```php
$users = DB::table('users')
    ->whereColumn('first_name', 'last_name')
    ->get();

$users = DB::table('users')
    ->whereColumn('updated_at', '>', 'created_at')
    ->get();
```

### Logical Grouping

```php
$users = DB::table('users')
    ->where('name', '=', 'John')
    ->where(function (Builder $query) {
        $query->where('votes', '>', 100)
            ->orWhere('title', '=', 'Admin');
    })
    ->get();

// Produces: SELECT * FROM users WHERE name = 'John' AND (votes > 100 OR title = 'Admin')
```

## Advanced Where Clauses

### Where Exists
```php
$users = DB::table('users')
    ->whereExists(function (Builder $query) {
        $query->select(DB::raw(1))
            ->from('orders')
            ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

### Subquery Where Clauses
```php
use App\Models\User;
use Illuminate\Database\Query\Builder;

$users = User::where(function (Builder $query) {
    $query->select('type')
        ->from('membership')
        ->whereColumn('membership.user_id', 'users.id')
        ->orderByDesc('membership.start_date')
        ->limit(1);
}, 'Pro')->get();
```

### Full Text Where Clauses (MariaDB, MySQL, PostgreSQL)
```php
$users = DB::table('users')
    ->whereFullText('bio', 'web developer')
    ->get();
```

## Ordering, Grouping, Limit and Offset

### Ordering

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->get();

// Multiple columns
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->orderBy('email', 'asc')
    ->get();

// Latest / oldest
$user = DB::table('users')
    ->latest()  // Orders by created_at DESC
    ->first();

// Random order
$randomUser = DB::table('users')
    ->inRandomOrder()
    ->first();
```

### Grouping

```php
$users = DB::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

### Limit and Offset

```php
$users = DB::table('users')
    ->offset(10)
    ->limit(5)
    ->get();
```

## Conditional Clauses

```php
$role = $request->input('role');

$users = DB::table('users')
    ->when($role, function (Builder $query, string $role) {
        $query->where('role_id', $role);
    })
    ->get();

// With else clause
$sortByVotes = $request->boolean('sort_by_votes');

$users = DB::table('users')
    ->when($sortByVotes, function (Builder $query, bool $sortByVotes) {
        $query->orderBy('votes');
    }, function (Builder $query) {
        $query->orderBy('name');
    })
    ->get();
```

## Insert Statements

```php
DB::table('users')->insert([
    'email' => '[email protected]',
    'votes' => 0
]);

// Insert multiple records
DB::table('users')->insert([
    ['email' => '[email protected]', 'votes' => 0],
    ['email' => '[email protected]', 'votes' => 0],
]);

// insertOrIgnore (ignores duplicate errors)
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => '[email protected]'],
    ['id' => 2, 'email' => '[email protected]'],
]);

// insertGetId (get auto-increment ID)
$id = DB::table('users')->insertGetId(
    ['email' => '[email protected]', 'votes' => 0]
);
```

### Upserts

```php
DB::table('flights')->upsert(
    [
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ],
    ['departure', 'destination'],  // Unique columns
    ['price']  // Columns to update
);
```

## Update Statements

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

### Update or Insert

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => '[email protected]', 'name' => 'John'],
        ['votes' => '2']
    );
```

### Updating JSON Columns
```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

### Increment and Decrement

```php
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');

// With additional columns
DB::table('users')->increment('votes', 1, ['name' => 'John']);

// Multiple columns
DB::table('users')->incrementEach([
    'votes' => 5,
    'balance' => 100,
]);
```

## Delete Statements

```php
$deleted = DB::table('users')->delete();
$deleted = DB::table('users')->where('votes', '>', 100)->delete();
```

## Pessimistic Locking

```php
// Shared lock
DB::table('users')
    ->where('votes', '>', 100)
    ->sharedLock()
    ->get();

// For update lock
DB::table('users')
    ->where('votes', '>', 100)
    ->lockForUpdate()
    ->get();
```

## Debugging

```php
// Dump and die
DB::table('users')->where('votes', '>', 100)->dd();

// Dump and continue
DB::table('users')->where('votes', '>', 100)->dump();

// Raw SQL with bindings substituted
DB::table('users')->where('votes', '>', 100)->dumpRawSql();
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/queries)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
