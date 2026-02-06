# Laravel 12.x Vite Documentation

## Introduction

Vite is a modern frontend build tool that provides fast development and production-ready asset bundling. Laravel integrates seamlessly with Vite through an official plugin.

## Installation

```bash
npm install
```

## Configuration

Create `vite.config.js`:

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

## Loading Scripts and Styles

In Blade templates:

```blade
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

## Running Vite

```bash
npm run dev    # Development with hot reload
npm run build  # Production build
```

## Working With Vue

```bash
npm install --save-dev @vitejs/plugin-vue
```

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

## Working With React

```bash
npm install --save-dev @vitejs/plugin-react
```

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

In Blade:

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

## Working With Inertia

```javascript
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
    resolve: (name) => resolvePageComponent(
        `./Pages/${name}.vue`,
        import.meta.glob('./Pages/**/*.vue')
    ),
    // ...
});
```

## Refreshing on Save

```javascript
export default defineConfig({
    plugins: [
        laravel({
            refresh: true,
        }),
    ],
});
```

## Processing Static Assets

```javascript
import.meta.glob([
    '../images/**',
    '../fonts/**',
]);
```

In Blade:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

## Asset Prefetching

```php
use Illuminate\Support\Facades\Vite;

public function boot(): void
{
    Vite::prefetch(concurrency: 3);
}
```

## Environment Variables

Prefix with `VITE_`:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

Access in JavaScript:

```javascript
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

## Disabling Vite in Tests

```php
test('example', function () {
    $this->withoutVite();
});
```

## Server-Side Rendering

```javascript
export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

```json
"scripts": {
    "build": "vite build && vite build --ssr"
}
```

## CSP Nonce

```php
use Illuminate\Support\Facades\Vite;

public function handle(Request $request, Closure $next): Response
{
    Vite::useCspNonce();
    return $next($request)->withHeaders([
        'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
    ]);
}
```

## Subresource Integrity

```bash
npm install --save-dev vite-plugin-manifest-sri
```

```javascript
import manifestSRI from 'vite-plugin-manifest-sri';

export default defineConfig({
    plugins: [
        laravel({ ... }),
        manifestSRI(),
    ],
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/vite)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
