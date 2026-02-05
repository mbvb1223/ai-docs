# Filament Widgets Documentation

## Overview

Filament enables developers to build dynamic dashboards using "widgets" — reusable UI elements that display data in specific formats. The framework supports statistics displays, charts, tables, and custom components.

## Creating Widgets

Use the command `php artisan make:filament-widget MyWidget` to generate a new widget. The command prompts selection from these types:

- **Custom**: Build from scratch
- **Chart**: Display chart data
- **Stats overview**: Show statistics
- **Table**: Render tabular data

## Widget Organization

Each widget class includes a `$sort` property to control display order:

```php
protected static ?int $sort = 2;
```

## Dashboard Customization

### Multiple Dashboards

Create multiple dashboard instances by extending the `Dashboard` class. Define routing with:

```php
protected static string $routePath = 'finance';
protected static ?string $title = 'Finance dashboard';
protected static ?int $navigationSort = 15;
```

### Grid Configuration

Override `getColumns()` to adjust layout. Responsive breakpoints are supported:

```php
public function getColumns(): int | array
{
    return [
        'md' => 4,
        'xl' => 5,
    ];
}
```

### Widget Width

Control widget span using the `$columnSpan` property (1-12 or 'full'):

```php
protected int | string | array $columnSpan = 'full';
```

Responsive widths use array format matching grid breakpoints.

## Conditional Visibility

Override the `canView()` method to show/hide widgets based on conditions:

```php
public static function canView(): bool
{
    return auth()->user()->isAdmin();
}
```

## Advanced Features

### Table Widgets

Generate table widgets with `php artisan make:filament-widget LatestOrders --table`, then customize using standard table configuration.

### Custom Widgets

Generated custom widgets are Livewire components with associated Blade views. Leverage full Livewire functionality and access component properties in templates via `$this`.

### Dashboard Filtering

Add filter forms using the `HasFiltersForm` trait. Widgets accessing filter data use the `InteractsWithPageFilters` trait with `$this->pageFilters` property access.

Alternative modal-based filtering uses `HasFiltersAction` with validated data submission.

### Persistence

Widget filters persist in user sessions by default. Disable via:

```php
protected bool $persistsFiltersInSession = false;
```

### Default Widgets

Disable default dashboard widgets in panel configuration:

```php
->widgets([])
```
