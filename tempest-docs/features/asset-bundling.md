# Asset Bundling

## Overview

Vite is the de-facto standard build tool for front-end development. It provides a very fast development server and bundles your assets for production without barely any configuration needed.

Tempest integrates with Vite through a Vite plugin and server-side package, streamlining frontend asset management in PHP applications.

## Quick Start

Install Vite using the installer command:

```bash
php tempest install vite
```

The wizard guides setup, including the Vite plugin, `vite.config.ts` configuration, TypeScript entrypoint, and optional Tailwind CSS integration.

Add the `<x-vite-tags />` component to your base template:

```html
<html lang="en">
	<head>
		<!-- ... -->
		<x-vite-tags />
	</head>
	<body>
		<x-slot />
	</body>
</html>
```

## Running the Development Server

During development, Vite transpiles assets on-the-fly for browser compatibility. Start the development server:

```bash
npm run dev
```

This executes the `dev` script from `package.json`, equivalent to running `npx vite`.

## Entrypoints

Files ending with `.entrypoint.{ts,css,js}` are automatically discovered and bundled:

```typescript
// app/main.entrypoint.ts
console.log('Hello, world!')
```

### Manually Including an Entrypoint

Include specific assets in particular views using the `entrypoint` attribute:

```html
<x-base>
	<slot name="head">
		<x-vite-tags entrypoint="src/Profile/profile.css" />
	</slot>
	<!-- ... -->
</x-base>
```

Note: Files must still be configured as entrypoints for production bundling.

### Manually Configuring Entrypoints

Create a `vite.config.php` file to specify entrypoints explicitly:

```php
return new ViteConfig(
    entrypoints: [
        'app/main.css',
        'app/main.ts',
    ],
);
```

Paths must be relative to the project root.

## Building for Production

Run the build script to bundle and version assets:

```bash
npm run build
```

Assets compile to the `public/build` directory by default. This should be added to `.gitignore`.

## Using a Nonce Attribute

For Content Security Policy compliance, add nonce attributes via route middleware:

```php
use Tempest\Support\Random;
use Tempest\Vite\ViteConfig;

final class ConfigureViteNonce implements HttpMiddleware
{
    public function __construct(
        private readonly ViteConfig $viteConfig,
    ) {}

    public function __invoke(Request $request, HttpMiddlewareCallable $next): Response
    {
        $this->viteConfig->nonce = Random\secure_string(length: 40);
        return $next($request);
    }
}
```

Register globally via event handler:

```php
#[EventHandler(KernelEvent::BOOTED)]
public function register(): void
{
    $this->router->addMiddleware(self::class);
}
```

## Subresource Integrity

Tempest automatically detects and adds subresource integrity hashes from `manifest.json`. Enable this using the `vite-plugin-manifest-sri` plugin:

```typescript
import sri from 'vite-plugin-manifest-sri'

export default defineConfig({
	plugins: [
		tailwindcss(),
		tempest(),
		sri(),
	],
})
```

## Testing

Tag generation is disabled during tests by default. Re-enable when needed:

```php
public function setUp(): void
{
    parent::setUp();
    $this->vite->allowTagResolution();
}
```
