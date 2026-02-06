# Laravel 12.x Frontend Documentation

## Introduction

Laravel is a backend framework providing features like routing, validation, caching, queues, and file storage. It offers developers a beautiful full-stack experience with powerful approaches for building application frontends.

There are two primary ways to build frontends with Laravel:
1. **Using PHP** - with Blade templating
2. **Using JavaScript Frameworks** - React or Vue with Inertia

---

## Using PHP

### PHP and Blade

Traditional PHP applications rendered HTML with simple templates and `echo` statements:

```php
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

**Laravel's Blade approach** provides a lightweight, convenient syntax:

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

#### Growing Expectations

As user expectations evolved, developers needed more dynamic frontends. Some solutions in the Laravel ecosystem include:
- **Laravel Livewire** - build modern UIs while staying in PHP
- **Alpine.js** - sprinkle JavaScript where needed

### Livewire

[Laravel Livewire](https://livewire.laravel.com) creates dynamic, modern frontends using PHP components that feel like modern JavaScript frameworks.

**Counter Component Example:**

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

**Corresponding Blade Template:**

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

Livewire enables custom HTML attributes (like `wire:click`) that connect your frontend and backend, all while using simple Blade expressions for state rendering.

**Recommendation:** Familiarize yourself with [views](../basics/views.md) and [Blade](../basics/blade.md), then explore the [official Livewire documentation](https://livewire.laravel.com/docs).

### Starter Kits

Use the [Livewire starter kit](starter-kits.md) to jump-start your application's development with PHP and Livewire.

---

## Using React or Vue

Many developers prefer JavaScript frameworks like React or Vue for their rich ecosystem and NPM packages. However, this creates challenges:
- Client-side routing
- Data hydration
- Authentication
- Maintaining two separate repositories

### Inertia

[Inertia](https://inertiajs.com) bridges Laravel and modern React/Vue frontends, allowing you to build full-featured applications within a **single code repository**.

**Controller Example:**

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

**React Component Example:**

```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```

**Benefits:**
- Leverage full power of React/Vue
- Use Laravel routes and controllers for routing, data hydration, and authentication
- Single code repository
- Full capability of both Laravel and React/Vue

#### Server-Side Rendering

Inertia offers [server-side rendering support](https://inertiajs.com/server-side-rendering). When deploying via [Laravel Cloud](https://cloud.laravel.com) or [Laravel Forge](https://forge.laravel.com), server-side rendering is easy to manage.

### Starter Kits

Use the [React or Vue application starter kits](starter-kits.md) to scaffold your application with:
- Backend and frontend authentication flow using Inertia
- Vue/React
- Tailwind CSS
- Vite

---

## Bundling Assets

Regardless of your frontend approach, you'll need to bundle CSS and JavaScript into production-ready assets.

### Vite

Laravel uses **[Vite](https://vitejs.dev)** by default, which provides:
- Lightning-fast build times
- Near-instantaneous Hot Module Replacement (HMR) during development

All new Laravel applications include a `vite.config.js` file with a lightweight Laravel Vite plugin.

**Quick Start:** Begin with [application starter kits](starter-kits.md), which include frontend and backend authentication scaffolding.

**For more details:** See the [dedicated Vite documentation](../basics/vite.md) on bundling and compiling assets.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/frontend)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
