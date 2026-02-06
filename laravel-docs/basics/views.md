# Laravel 12.x Views Documentation

## Introduction

Views separate your HTML from your application logic and are stored in `resources/views`. Laravel views are typically written using Blade templating.

## Creating and Rendering Views

### Creating a View

```bash
php artisan make:view greeting
```

### Rendering Views

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});

// Using facade
use Illuminate\Support\Facades\View;
return View::make('greeting', ['name' => 'James']);
```

### Nested View Directories

```php
return view('admin.profile', $data);
// References: resources/views/admin/profile.blade.php
```

### First Available View

```php
return View::first(['custom.admin', 'admin'], $data);
```

### Check if View Exists

```php
if (View::exists('admin.profile')) {
    // ...
}
```

## Passing Data to Views

### Basic Data Passing

```php
return view('greetings', ['name' => 'Victoria']);
```

### Using with() Method

```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

### Sharing Data With All Views

In a service provider:

```php
use Illuminate\Support\Facades\View;

public function boot(): void
{
    View::share('key', 'value');
}
```

## View Composers

View composers are callbacks executed when a view is rendered, useful for binding data to views.

### Registering View Composers

```php
use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;

public function boot(): void
{
    // Class-based
    View::composer('profile', ProfileComposer::class);

    // Closure-based
    View::composer('welcome', function (View $view) {
        // ...
    });
}
```

### Creating a Composer Class

```php
namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    public function __construct(
        protected UserRepository $users,
    ) {}

    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

### Multiple Views

```php
View::composer(['profile', 'dashboard'], MultiComposer::class);
```

### Wildcard Composers

```php
View::composer('*', function (View $view) {
    // Applied to all views
});
```

## View Creators

View creators execute immediately after view instantiation (before render):

```php
View::creator('profile', ProfileCreator::class);
```

## Optimizing Views

### Precompile Views

```bash
php artisan view:cache
```

### Clear View Cache

```bash
php artisan view:clear
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/views)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
