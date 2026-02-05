# View Specifications

## Overview

Tempest View is a server-side templating engine powered by PHP, with syntax inspired by Vue.js. It aims to stay as close as possible to HTML, using PHP where needed.

## Basic Syntax

### Expression Attributes

Attributes prefixed with `:` are interpreted as PHP code:

```php
<div :if="$condition"></div>
<x-component :title="$content->title"></x-component>
```

### Escaped Expression Attributes

Use `::` to escape attributes for frontend frameworks:

```php
<div ::if="frontend-code"></div>
```

### Control Structures

Supported directives: `:if`, `:elseif`, `:else`, `:isset`, `:foreach`, `:forelse`

```php
<div :if="$condition">A</div>
<div :elseif="$otherCondition">B</div>
<div :else>C</div>
```

Loops with fallback content:

```php
<div :foreach="$items as $key => $item">Item</div>
<div :forelse>Nothing here</div>
```

### Combined Control Structures

Multiple directives work together in evaluation order:

```php
<div :foreach="$items as $key => $item" :if="$key !== 0">
    <!-- Never print the first item -->
</div>
```

### Echoing Data

- `{{ $var }}` - escaped output
- `{!! $raw !!}` - raw output
- Both support PHP expressions

```php
{{ strtoupper($var) }}
{!! $markdown->render($content) !!}
```

### Comments

Server-side comments (stripped from output):

```php
{{-- This won't appear in HTML --}}
```

Client-side comments (visible to browser):

```php
<!-- This appears in HTML -->
```

### Imports

Views automatically merge imports, allowing per-view dependencies:

```php
<?php
use App\PostController;
use function Tempest\Router\uri;
?>
```

### View File Resolution

Returns views with named arguments:

```php
return view(__DIR__ . '/views/home.view.php', title: 'foo');
return view('./views/home.view.php', title: 'foo');
return view('views/home.view.php', title: 'foo');
```

Resolution follows this priority:
1. Absolute paths as-is
2. Relative to controller location
3. Discovery locations

View files must end with `.view.php`

### View Objects

Custom view objects implement the `View` interface:

```php
use Tempest\View\View;
use Tempest\View\IsView;

final class BookView implements View
{
    use IsView;

    public function __construct(
        public string $title,
        public Book $book,
    ) {
        $this->path = __DIR__ . '/books.view.php';
    }

    public function summarize(Book $book): string
    {
        return // ...
    }
}
```

Public properties and methods are exposed to views.

### Templates

The `<x-template>` element applies directives without rendering a DOM node:

```php
<x-template :foreach="$posts as $post">
    <div>{{ $post->title }}</div>
</x-template>
```

Renders only the children, not a wrapper.

### Boolean Attributes

Expression attributes returning booleans follow HTML specification—attributes omit when falsy:

```php
<option :value="$value" :selected="$selected">{{ $label }}</option>
```

Also works with non-standard attributes:

```php
<div :data-active="{$isActive}"></div>
<!-- <div></div> when falsy -->
<!-- <div data-active></div> when truthy -->
```

## View Components

Files starting with `x-` become components (e.g., `x-button.view.php`).

### Registering Components

Anonymous components auto-discover from `.view.php` files:

```php
<!-- app/x-base.view.php -->
<html lang="en">
    <head>
        <title :if="$title">{{ $title }} — AirAcme</title>
        <title :else>AirAcme</title>
    </head>
    <body>
        <x-slot />
    </body>
</html>
```

### Using Components

Reference by name with the `x-` prefix:

```php
<x-base :title="$this->post->title">
    <article>
        {{ $this->post->body }}
    </article>
</x-base>
```

### Attributes in Components

Passed attributes become variables. Casing affects naming:

- `camelCase`/`PascalCase` → `$lowercase`
- `kebab-case`/`snake_case` → `$camelCase`

Idiomatic approach uses `kebab-case`.

### Fallthrough Attributes

`class` and `style` automatically merge with root node; `id` replaces existing:

```php
<!-- x-button.view.php -->
<button class="rounded-md px-2.5 py-1.5 text-sm">...</button>

<!-- Usage -->
<x-button class="text-gray-100 bg-gray-900" />
<!-- Classes merge -->
```

### Dynamic Attributes

Access all passed attributes via `$attributes` array (kebab-case naming):

```php
<!-- x-badge.view.php -->
<span class="px-2 py-1 rounded-md text-sm bg-gray-100 text-gray-900">
    {{ $attributes['value'] }}
</span>
```

### Using Slots

Define content outlets with `<x-slot />`:

```php
<!-- x-button.view.php -->
<button class="rounded-md px-2.5 py-1.5 text-sm text-gray-100 bg-gray-900">
    <x-slot />
</button>

<!-- Usage -->
<x-button>
    <x-icon name="tabler:x" />
    <span>Delete</span>
</x-button>
```

### Default Slot Content

Slots support fallback values:

```php
<div>
    <x-slot>Fallback value</x-slot>
    <x-slot name="a">Fallback for named slot</x-slot>
</div>
```

### Named Slots

Multiple outlets via `name` attribute:

```php
<!-- x-base.view.php -->
<html>
    <head>
        <x-slot name="styles" />
    </head>
    <body>
        <x-slot />
    </body>
</html>

<!-- Usage -->
<x-base title="Hello World">
    <x-slot name="styles">
        <style>body { /* ... */ }</style>
    </x-slot>
    <p>Hello World</p>
</x-base>
```

### Dynamic Slots

Access slots via `$slots` variable (instance of `Tempest\View\Slot`):

Properties:
- `$slot->name` - slot's name
- `$slot->content` - compiled content
- `$slot->attributes` - defined attributes
- `$slot->{attribute}` - dynamic attribute access

Example tab component:

```php
<!-- x-tabs.view.php -->
<div :foreach="$slots as $slot">
    <h1 :title="$slot->title">{{ $slot->name }}</h1>
    <p>{!! $slot->content !!}</p>
</div>

<!-- Usage -->
<x-tabs>
    <x-slot name="php" title="PHP">This is the PHP tab</x-slot>
    <x-slot name="js" title="JavaScript">This is the JavaScript tab</x-slot>
</x-tabs>
```

### Dynamic View Components

Render components by runtime-determined names:

```php
<!-- $name = 'x-post' -->
<x-component :is="$name" :title="$title" />
```

### View Component Scope

Components act like PHP closures—variables require explicit passing. View-defined data via controller is directly accessible:

```php
<!-- HomeController -->
final class HomeController
{
    #[Get('/')]
    public function __invoke(): View
    {
        return view('<x-base />', siteTitle: 'Tempest');
    }
}

<!-- x-base.view.php -->
<h1>{{ $siteTitle }}</h1>
```

## Built-in Components

Metadata available via hidden command:

```bash
./tempest meta:view-component [view-component]
```

### x-base

Base template with Tailwind CDN for prototyping:

```php
<x-base :title="Blog">
    <h1>Welcome!</h1>
</x-base>
```

### x-form

Form element with CSRF token included:

```php
<x-form :action="uri(StorePostController::class)">
    <!-- ... -->
</x-form>
```

### x-input

Versatile input with automatic labels and validation errors:

```php
<x-input name="title" />
<x-input name="content" type="textarea" label="Write your content" />
<x-input name="email" type="email" id="other_email" />
```

### x-submit

Submit button (defaults to "Submit" label):

```php
<x-submit />
<x-submit label="Send" />
```

### x-csrf-token

Includes CSRF token in forms:

```php
<form action="...">
    <x-csrf-token />
</form>
```

### x-icon

Injects icons from Iconify project:

```php
<x-icon name="material-symbols:php" class="size-4 text-indigo-400" />
```

Icons are fetched from Iconify API on first use, then cached indefinitely.

### x-vite-tags

Integrates Vite entrypoints:

```php
<!-- x-base.view.php -->
<html lang="en">
    <head>
        <x-vite-tags />
    </head>
</html>
```

Optional `entrypoint` attribute limits injection to specific entries.

### x-template

See Templates section.

### x-slot

See Using Slots section.

### x-markdown

Renders markdown content:

```php
<x-markdown># hi</x-markdown>
<x-markdown :content="$text" />
```

### x-component

Reserved component for dynamic rendering:

```php
<x-component is="x-post" :title="$title">
    Content
</x-component>
```
