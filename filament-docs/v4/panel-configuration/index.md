# Panel Configuration - Filament Documentation

## Introduction

The Filament panel configuration file is typically located at `app/Providers/Filament/AdminPanelProvider.php`. This file controls the setup for the default `/admin` panel, though you can create additional panels as needed.

## Creating and Managing Panels

### Default Admin Panel

When you install Filament using `filament:install`, a configuration file is created that handles the `/admin` panel setup. All resources, custom pages, and dashboard widgets register to this panel by default.

### Creating Additional Panels

To create a new panel, use the artisan command:

```bash
php artisan make:filament-panel app
```

This generates a provider at `app/Providers/Filament/AppPanelProvider.php` and makes the panel accessible at `/app`. The new provider must be registered in `bootstrap/providers.php` (Laravel 11+) or `config/app.php` (Laravel 10).

## Key Configuration Options

### Path Configuration

Customize the URL path where your panel is accessible:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->path('app');
}
```

Use an empty string to serve at the root—ensure no conflicting routes exist.

### Domain Binding

Restrict a panel to a specific domain:

```php
->domain('admin.example.com');
```

### Content Width

Set maximum content width using the `maxContentWidth()` method with Tailwind scale options. The default is `SevenExtraLarge`. For simple pages (login/registration), use `simplePageMaxContentWidth()`:

```php
use Filament\Support\Enums\Width;

->maxContentWidth(Width::Full)
->simplePageMaxContentWidth(Width::Small)
```

### Sub-Navigation Position

Control sub-navigation placement across the panel:

```php
use Filament\Pages\Enums\SubNavigationPosition;

->subNavigationPosition(SubNavigationPosition::End)
```

Options include `Start`, `End`, or `Top` (renders as tabs).

## Advanced Features

### SPA Mode

Enable single-page-application behavior using Livewire's `wire:navigate`:

```php
->spa()
```

With prefetching enabled:

```php
->spa(hasPrefetching: true)
```

Exclude specific URLs from SPA navigation:

```php
->spaUrlExceptions([
    '*/admin/posts/*',
])
```

### Unsaved Changes Alerts

Warn users before navigating away with unsaved changes:

```php
->unsavedChangesAlerts()
```

### Database Transactions

Enable automatic database transaction wrapping for all operations:

```php
->databaseTransactions()
```

Disable for specific actions using `databaseTransaction(false)` or opt in selectively.

### Render Hooks

Register panel-specific Blade content at various lifecycle points:

```php
use Filament\View\PanelsRenderHook;

->renderHook(
    PanelsRenderHook::BODY_START,
    fn (): string => Blade::render('@livewire(\'modal\')'),
)
```

### Middleware

Apply middleware to all routes:

```php
->middleware([
    CustomMiddleware::class,
], isPersistent: true)
```

For authenticated routes only:

```php
->authMiddleware([
    VerifyTenant::class,
])
```

### Asset Registration

Register CSS and JavaScript specific to a panel:

```php
use Filament\Support\Assets\Css;

->assets([
    Css::make('custom', resource_path('css/custom.css')),
])
```

### Authorization

Enable strict authorization mode to throw exceptions when policies don't exist:

```php
->strictAuthorization()
```

### Error Notifications

Customize error handling in production (debug mode off):

```php
->registerErrorNotification(
    title: 'An error occurred',
    body: 'Please try again later.',
    statusCode: 404,
)
```

Disable error notifications:

```php
->errorNotifications(false)
```

### Broadcasting

Disable automatic Laravel Echo connection:

```php
->broadcasting(false)
```

### Lifecycle Hooks

Execute code during panel initialization:

```php
->bootUsing(function (Panel $panel) {
    // Custom setup logic
})
```
