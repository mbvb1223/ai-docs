# Filament Installation Guide

## System Requirements

Filament requires:
- PHP 8.2+
- Laravel v11.28+
- Tailwind CSS v4.1+

## Installation Options

### Panel Builder Installation

For building admin panels or dashboards, install via Composer:

```bash
composer require filament/filament:"^4.0"
php artisan filament:install --panels
```

**Windows PowerShell users** should use the tilde constraint instead:
```bash
composer require filament/filament:"~4.0"
php artisan filament:install --panels
```

This creates an `AdminPanelProvider` service provider. Verify it's registered in `bootstrap/providers.php`.

Create a user account:
```bash
php artisan make:filament-user
```

Access your panel at `/admin` in your browser.

### Individual Components Installation

Install only the packages you need:

```bash
composer require \
    filament/tables:"^4.0" \
    filament/schemas:"^4.0" \
    filament/forms:"^4.0" \
    filament/infolists:"^4.0" \
    filament/actions:"^4.0" \
    filament/notifications:"^4.0" \
    filament/widgets:"^4.0"
```

For Blade UI components only, require `filament/support`.

## Frontend Setup

### For New Projects

```bash
php artisan filament:install --scaffold
npm install
npm run dev
```

### For Existing Projects

Install Tailwind CSS:
```bash
npm install tailwindcss @tailwindcss/vite --save-dev
```

**Import CSS** in `resources/css/app.css`:

```css
@import 'tailwindcss';
@import '../../vendor/filament/support/resources/css/index.css';
@import '../../vendor/filament/actions/resources/css/index.css';
@import '../../vendor/filament/forms/resources/css/index.css';
@import '../../vendor/filament/infolists/resources/css/index.css';
@import '../../vendor/filament/notifications/resources/css/index.css';
@import '../../vendor/filament/schemas/resources/css/index.css';
@import '../../vendor/filament/tables/resources/css/index.css';
@import '../../vendor/filament/widgets/resources/css/index.css';

@variant dark (&:where(.dark, .dark *));
```

**Configure Vite** (`vite.config.js`):

```javascript
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
})
```

Compile assets:
```bash
npm run dev
```

### Layout Configuration

Create your Blade layout at `resources/views/components/layouts/app.blade.php`:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="application-name" content="{{ config('app.name') }}">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{{ config('app.name') }}</title>
        <style>
            [x-cloak] { display: none !important; }
        </style>
        @filamentStyles
        @vite('resources/css/app.css')
    </head>
    <body class="antialiased">
        {{ $slot }}
        @livewire('notifications')
        @filamentScripts
        @vite('resources/js/app.js')
    </body>
</html>
```

Include `@filamentStyles` in the head and `@filamentScripts` at the end of the body tag.

## Configuration Publishing

Publish the configuration file:

```bash
php artisan vendor:publish --tag=filament-config
```

This creates `config/filament.php` where you can customize settings across all Filament packages.
