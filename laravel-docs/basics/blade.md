# Blade Templates - Laravel 12.x Documentation

## Introduction

Blade is Laravel's simple yet powerful templating engine included with the framework. Unlike some PHP templating engines, Blade doesn't restrict you from using plain PHP code in templates. All Blade templates are compiled into plain PHP code and cached until modified, adding essentially zero overhead to your application.

Blade template files use the `.blade.php` extension and are typically stored in the `resources/views` directory.

### Supercharging Blade With Livewire

For dynamic interfaces and reactive frontends, consider using [Laravel Livewire](https://livewire.laravel.com), which augments Blade components with dynamic functionality without the complexities of JavaScript frameworks.

## Displaying Data

Display data passed to Blade views by wrapping variables in curly braces:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

```blade
Hello, {{ $name }}.
```

Blade's `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks.

### Displaying Unescaped Data

Use the `{!! !!}` syntax to display unescaped data:

```blade
Hello, {!! $name !!}.
```

**Warning:** Only use unescaped syntax with trusted data to prevent XSS attacks.

### Blade and JavaScript Frameworks

Use the `@` symbol to prevent Blade from processing an expression:

```blade
Hello, @{{ name }}.
```

### Rendering JSON

Use `Illuminate\Support\Js::from()` to safely render JSON:

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Or use the `Js` facade:

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

### The `@verbatim` Directive

Wrap HTML sections to avoid prefixing each echo statement with `@`:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Blade Directives

### If Statements

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Additional shortcuts:

```blade
@unless (Auth::check())
    You are not signed in.
@endunless

@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### Authentication Directives

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest

@auth('admin')
    // Check specific guard...
@endauth
```

#### Environment Directives

```blade
@production
    // Production specific content...
@endproduction

@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // Multiple environments...
@endenv
```

### Switch Statements

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### Loops

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

#### Loop Control

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

### The Loop Variable

Inside `@foreach`, a `$loop` variable provides loop information:

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

**Loop Variable Properties:**

| Property | Description |
|----------|-------------|
| `$loop->index` | Current loop iteration index (starts at 0) |
| `$loop->iteration` | Current loop iteration (starts at 1) |
| `$loop->remaining` | Iterations remaining |
| `$loop->count` | Total items in array |
| `$loop->first` | First iteration |
| `$loop->last` | Last iteration |
| `$loop->even` | Even iteration |
| `$loop->odd` | Odd iteration |
| `$loop->depth` | Nesting level |
| `$loop->parent` | Parent loop variable |

### Conditional Classes & Styles

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>
```

Conditional styles:

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>
```

### Additional Attributes

```blade
<input type="checkbox" name="active" value="active" @checked(old('active', $user->active)) />

<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>

<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>

<input type="email" name="email" value="email@example.com" @readonly($user->isNotAdmin()) />

<input type="text" name="title" value="title" @required($user->isAdmin()) />
```

### Including Subviews

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Pass additional data:

```blade
@include('view.name', ['status' => 'complete'])

@includeIf('view.name', ['status' => 'complete'])

@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])

@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

#### Rendering Views for Collections

```blade
@each('view.name', $jobs, 'job')

@each('view.name', $jobs, 'job', 'view.empty')
```

### The `@once` Directive

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce

@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

### Raw PHP

```blade
@php
    $counter = 1;
@endphp
```

Import classes:

```blade
@use('App\Models\Flight')

@use('App\Models\Flight', 'FlightModel')
```

### Comments

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

## Components

### Creating Components

Use Artisan to create a class-based component:

```bash
php artisan make:component Alert

php artisan make:component Forms/Input
```

### Rendering Components

```blade
<x-alert/>

<x-user-profile/>

<x-inputs.button/>
```

### Passing Data to Components

```blade
<x-alert type="error" :message="$message"/>
```

Define a component class:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    public function render(): View
    {
        return view('components.alert');
    }
}
```

Component view:

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### Short Attribute Syntax

```blade
{{-- Short syntax --}}
<x-profile :$userId :$name />

{{-- Equivalent to --}}
<x-profile :user-id="$userId" :name="$name" />
```

### Component Attributes

Attributes not part of the constructor are added to the attribute bag:

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

#### Merging Attributes

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Conditional class merging:

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

### Slots

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Use the component:

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Named slots:

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

```blade
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

### Anonymous Components

Create via `--view` flag:

```bash
php artisan make:component forms.input --view
```

Creates `resources/views/components/forms/input.blade.php`:

```blade
<x-forms.input/>
```

### Data Properties

Use `@props` to define component variables:

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

### Dynamic Components

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

## Building Layouts

### Layouts Using Components

Define a layout component:

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

Use the layout:

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

### Layouts Using Template Inheritance

Define a layout:

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Extend the layout:

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

## Forms

### CSRF Field

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

### Method Field

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

### Validation Errors

```blade
<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## Stacks

Push to named stacks:

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Render stack:

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

Prepend to stack:

```blade
@prepend('scripts')
    This will be first...
@endprepend
```

## Service Injection

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## Rendering Inline Blade Templates

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

## Rendering Blade Fragments

Define fragments:

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

Render specific fragment:

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

## Extending Blade

### Custom Directives

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

Use as:

```blade
@datetime($var)
```

### Custom If Statements

```php
Blade::if('disk', function ($value) {
    return config('filesystems.default') === $value;
});
```

Use as:

```blade
@disk('local')
    <!-- Local disks -->
@elsedisk('s3')
    <!-- S3 disks -->
@else
    <!-- Other disks -->
@enddisk
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/blade)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
