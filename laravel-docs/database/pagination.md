# Laravel 12.x Database Pagination Documentation

## Introduction

Laravel's paginator is integrated with the query builder and Eloquent ORM, providing convenient pagination with zero configuration. HTML is Tailwind CSS compatible by default, with Bootstrap support also available.

## Basic Usage

### Paginating Query Builder Results

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

### Simple Pagination

For efficient queries without total page count:

```php
$users = DB::table('users')->simplePaginate(15);
```

### Paginating Eloquent Results

```php
use App\Models\User;

$users = User::paginate(15);
$users = User::where('votes', '>', 100)->paginate(15);
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

### Cursor Pagination

Better performance for large datasets:

```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

**Cursor advantages:**
- Better performance for large datasets
- Handles frequent writes better

**Cursor limitations:**
- Only displays "Next" and "Previous" links
- Requires unique columns in ordering

### Multiple Paginator Instances

```php
$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

### Customizing Pagination URLs

```php
$users = User::paginate(15);
$users->withPath('/admin/users');
$users->appends(['sort' => 'votes']);
$users->withQueryString();
$users->fragment('users');
```

## Displaying Pagination Results

### Basic Display

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

### Adjusting Link Window

```blade
{{ $users->onEachSide(5)->links() }}
```

### Converting to JSON

```php
Route::get('/users', function () {
    return User::paginate();
});
```

## Customizing the Pagination View

### Using a Custom View

```blade
{{ $paginator->links('view.name') }}
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

### Publishing Pagination Views

```bash
php artisan vendor:publish --tag=laravel-pagination
```

### Setting Default View

```php
use Illuminate\Pagination\Paginator;

public function boot(): void
{
    Paginator::defaultView('view-name');
    Paginator::defaultSimpleView('view-name');
}
```

### Using Bootstrap

```php
use Illuminate\Pagination\Paginator;

public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

## Paginator Instance Methods

| Method | Description |
|--------|-------------|
| `count()` | Items for current page |
| `currentPage()` | Current page number |
| `firstItem()` | Result number of first item |
| `hasPages()` | Multiple pages exist |
| `hasMorePages()` | More items exist |
| `items()` | Items for current page |
| `lastItem()` | Result number of last item |
| `lastPage()` | Last page number |
| `nextPageUrl()` | URL for next page |
| `onFirstPage()` | On first page |
| `onLastPage()` | On last page |
| `perPage()` | Items per page |
| `previousPageUrl()` | URL for previous page |
| `total()` | Total matching items |
| `url($page)` | URL for given page |
| `through($callback)` | Transform each item |

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/pagination)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
