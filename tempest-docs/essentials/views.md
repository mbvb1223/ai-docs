# Views

## Overview

Tempest provides a modern templating engine inspired by front-end frameworks. The syntax extends HTML with PHP-like features, though Blade, Twig, or other engines can be substituted if preferred.

## Rendering Views

Views are returned from controller actions using the `view()` function, which instantiates a `Tempest\View\View` object:

```php
use Tempest\View\View;
use function Tempest\View\view;

final readonly class AircraftController {
    #[Get(uri: '/aircraft/{aircraft}')]
    public function show(Aircraft $aircraft): View {
        return view('aircraft.view.php', aircraft: $aircraft);
    }
}
```

### View Paths

View paths can be relative or absolute. These are equivalent:
- `view(__DIR__ . '/views/home.view.php')`
- `view('./views/home.view.php')`
- `view('views/home.view.php')`

### Dedicated View Objects

View objects are dedicated classes representing specific views, improving static analysis:

```php
final class AircraftView implements View {
    use IsView;

    public function __construct(
        public Aircraft $aircraft,
        public AircraftType $type,
    ) {
        $this->path = root_path('src/Aircraft/aircraft.view.php');
    }
}
```

## Templating Syntax

### Text Interpolation

Escaped output uses mustache syntax:
```html
<span>Welcome, {{ $username }}</span>
```

Unescaped output (use only with sanitized data):
```html
<div>{!! $content !!}</div>
```

### Expression Attributes

HTML attributes evaluated as PHP code use colon prefix:
```html
<html :lang="$this->user->language"></html>
```

Only scalar and `Stringable` values work on HTML elements; components accept any object.

### Boolean Attributes

Boolean attributes follow HTML specification, not rendering if false:
```html
<option :value="$value" :selected="$selected">{{ $label }}</option>
```

## Control Flow Directives

### If/Else

```html
<span :if="$this->pendingUploads->isEmpty()">Import files</span>
<span :else>Import {{ $this->pendingUploads->count() }} file(s)</span>
```

### Isset

```html
<h1 :isset="$title">{{ $title }}</h1>
<h1 :else>Title</h1>
```

### Foreach/Forelse

```html
<li :foreach="$this->reports as $report">
  {{ $report->title }}
</li>
<li :forelse>
    There is no report.
</li>
```

### Templates

The `<x-template>` element renders directives without adding DOM elements:
```html
<x-template :foreach="$posts as $post">
    <div>{{ $post->title }}</div>
</x-template>
```

## View Components

Components split UI into independent, reusable pieces.

### Registering Components

Create `.view.php` files starting with `x-`. These anonymous components are auto-discovered:

```php
// app/x-base.view.php
<html lang="en">
    <head>
        <title :if="$title ?? null">{{ $title }} — AirAcme</title>
        <title :else>AirAcme</title>
    </head>
    <body>
        <x-slot />
    </body>
</html>
```

### Using Components

```html
<x-base :title="$this->post->title">
    <article>{{ $this->post->body }}</article>
</x-base>
```

### Attributes in Components

Attributes become variables in the component. Casing conventions:
- `camelCase`/`PascalCase` → `$lowercase`
- `kebab-case`/`snake_case` → `$camelCase`

### Fallthrough Attributes

`class` and `style` attributes merge with root node attributes. The `id` attribute replaces existing values.

### Dynamic Attributes

Access all attributes (except expression attributes) via `$attributes` array using `kebab-case` keys:

```php
// x-badge.view.php
<span class="px-2 py-1 rounded-md text-sm bg-gray-100">
    {{ $attributes['value'] }}
</span>
```

## Slots

### Default Slots

```html
<!-- x-button.view.php -->
<button class="rounded-md px-2.5 py-1.5 text-sm text-gray-100 bg-gray-900">
    <x-slot />
</button>
```

### Default Slot Content

```html
<div>
    <x-slot>Fallback value</x-slot>
    <x-slot name="a">Fallback for named slot</x-slot>
</div>
```

### Named Slots

```html
<!-- x-base.view.php -->
<head>
    <x-slot name="styles" />
</head>
<body>
    <x-slot />
</body>

<!-- Usage -->
<x-base title="Hello World">
    <x-slot name="styles">
        <style>/* ... */</style>
    </x-slot>
    <p>Hello World</p>
</x-base>
```

### Dynamic Slots

Within components, `$slots` provides access to named slots:

```php
// x-tabs.view.php
<div :foreach="$slots as $slot">
    <h1 :title="$slot->title">{{ $slot->name }}</h1>
    <p>{!! $slot->content !!}</p>
</div>
```

### Dynamic View Components

Render components by name at runtime:
```html
<x-component :is="$name" :title="$title" />
```

### Component Scope

Components access only explicitly-provided variables (like closures). View-defined variables are available as local variables:

```php
final class HomeController {
    #[Get('/')]
    public function __invoke(): View {
        return view('<x-base />', siteTitle: 'Tempest');
    }
}

// x-base.view.php
<h1>{{ $siteTitle }}</h1>
```

## Built-in Components

Install via `tempest install view-components`:

- **`x-base`**: Base template with Tailwind CDN
- **`x-form`**: Form with CSRF token included
- **`x-input`**: Input with labels and validation errors
- **`x-submit`**: Submit button with "Submit" default label
- **`x-csrf-token`**: CSRF token for forms
- **`x-icon`**: Icons from Iconify project
- **`x-vite-tags`**: Injects Vite entrypoints
- **`x-markdown`**: Renders markdown content

Example:
```html
<x-form :action="uri(StorePostController::class)">
    <x-input name="title" />
    <x-input name="content" type="textarea" />
    <x-submit />
</x-form>
```

## Pre-processing Views

Implement `Tempest\View\ViewProcessor` to provide common data:

```php
final class StarCountViewProcessor implements ViewProcessor {
    public function __construct(private readonly GitHub $github) {}

    public function process(View $view): View {
        if (!$view instanceof WithStargazersCount) {
            return $view;
        }
        return $view->data(stargazers: $this->github->getStarCount());
    }
}
```

## View Caching

Enable caching in production via `.env`:
```
VIEW_CACHE=true
```

Clear cache with `tempest view:clear` during deployments.

## Standalone Usage

Use Tempest View without the full framework:

```php
use Tempest\View\Renderers\TempestViewRenderer;

$renderer = TempestViewRenderer::make();
$html = $renderer->render(view('home.view.php', name: 'Brent'));
```

With component support:
```php
$config = new ViewConfig()->addViewComponents(
    __DIR__ . '/components/x-base.view.php',
    __DIR__ . '/components/x-footer.view.php',
);
$renderer = TempestViewRenderer::make($config);
```

## Separate View Directories

Add custom namespaces in `composer.json`:
```json
"autoload": {
    "psr-4": {
        "App\\": "src/",
        "Views\\": "views/"
    }
}
```

Run `composer update` after changes.

## Using Other Engines

### Twig

```bash
composer require twig/twig
```

Configure in `twig.config.php`:
```php
return new TwigConfig(
    viewPaths: [__DIR__ . '/views/'],
);
```

Update `view.config.php`:
```php
return new ViewConfig(
    rendererClass: \Tempest\View\Renderers\TwigViewRenderer::class,
);
```

### Blade

```bash
composer require tempest/blade
```

Configure in `blade.config.php`:
```php
return new BladeConfig(
    viewPaths: [__DIR__ . '/views/'],
);
```

Update `view.config.php`:
```php
return new ViewConfig(
    rendererClass: \Tempest\View\Renderers\BladeViewRenderer::class,
);
```

### Custom Renderers

Implement `Tempest\View\ViewRenderer`:

```php
final readonly class CustomRenderer implements ViewRenderer {
    public function render(View|string|null $view): string {
        // Implementation
    }
}
```

Register via `ViewConfig`:
```php
return new ViewConfig(
    rendererClass: CustomRenderer::class,
);
```

Use singleton initializers for engines requiring setup.
