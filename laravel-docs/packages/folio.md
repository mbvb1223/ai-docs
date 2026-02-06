# Laravel 12.x Folio Documentation

## Introduction

Laravel Folio is a page-based router - create Blade templates in `resources/views/pages` and routes are automatically generated.

## Installation

```bash
composer require laravel/folio
php artisan folio:install
```

## Creating Routes

### Basic Pages

```bash
php artisan folio:page schedule
# pages/schedule.blade.php → /schedule

php artisan folio:page user/profile
# pages/user/profile.blade.php → /user/profile
```

### Index Routes

```bash
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

## Route Parameters

```bash
php artisan folio:page "users/[id]"
# pages/users/[id].blade.php → /users/1
```

```blade
<div>
    User {{ $id }}
</div>
```

### Multiple Segments

```bash
php artisan folio:page "users/[...ids]"
```

```blade
@foreach ($ids as $id)
    <li>User {{ $id }}</li>
@endforeach
```

## Route Model Binding

```bash
php artisan folio:page "users/[User]"
```

```blade
<div>
    User {{ $user->id }}
</div>
```

### Custom Keys

```bash
php artisan folio:page "[Post:slug].blade.php"
```

## Named Routes

```blade
<?php

use function Laravel\Folio\name;

name('users.index');
```

```blade
<a href="{{ route('users.index') }}">All Users</a>
```

## Middleware

### Per-Page

```blade
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);
```

### Path-Based

```php
Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',
    ],
]);
```

## Render Hooks

```blade
<?php

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
});
```

## Configuration

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware(['auth']);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/folio)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
