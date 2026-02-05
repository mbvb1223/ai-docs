# Stats Overview Widgets - Filament Documentation

## Introduction

Filament provides a built-in "stats overview" widget template for displaying multiple statistics without custom view code.

Create a widget using:
```bash
php artisan make:filament-widget StatsOverview --stats-overview
```

Return `Stat` instances from the `getStats()` method:

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Unique views', '192.1k'),
            Stat::make('Bounce rate', '21%'),
            Stat::make('Average time on page', '3:12'),
        ];
    }
}
```

## Adding Descriptions and Icons

Enhance stats with descriptive text and icons:

```php
use Filament\Widgets\StatsOverviewWidget\Stat;

protected function getStats(): array
{
    return [
        Stat::make('Unique views', '192.1k')
            ->description('32k increase')
            ->descriptionIcon('heroicon-m-arrow-trending-up'),
        Stat::make('Bounce rate', '21%')
            ->description('7% decrease')
            ->descriptionIcon('heroicon-m-arrow-trending-down'),
    ];
}
```

Position icons before the description using `IconPosition::Before`:

```php
use Filament\Support\Enums\IconPosition;

Stat::make('Unique views', '192.1k')
    ->description('32k increase')
    ->descriptionIcon('heroicon-m-arrow-trending-up', IconPosition::Before)
```

## Color Customization

Apply colors to statistics:

```php
Stat::make('Unique views', '192.1k')
    ->description('32k increase')
    ->descriptionIcon('heroicon-m-arrow-trending-up')
    ->color('success')
```

## HTML Attributes

Add custom HTML attributes using `extraAttributes()`:

```php
Stat::make('Processed', '192.1k')
    ->color('success')
    ->extraAttributes([
        'class' => 'cursor-pointer',
        'wire:click' => "\$dispatch('setStatusFilter', { filter: 'processed' })",
    ])
```

## Chart Integration

Include historical data visualization:

```php
Stat::make('Unique views', '192.1k')
    ->description('32k increase')
    ->descriptionIcon('heroicon-m-arrow-trending-up')
    ->chart([7, 2, 10, 3, 15, 4, 17])
    ->color('success')
```

## Polling Configuration

Refresh stats every 5 seconds by default. Customize the interval:

```php
protected ?string $pollingInterval = '10s';
```

Disable polling entirely:

```php
protected ?string $pollingInterval = null;
```

## Lazy Loading

Widgets load only when visible. Disable this behavior:

```php
protected static bool $isLazy = false;
```

## Headers and Descriptions

Add contextual text above the widget:

```php
protected ?string $heading = 'Analytics';
protected ?string $description = 'An overview of some analytics.';
```

Or use dynamic methods:

```php
protected function getHeading(): ?string
{
    return 'Analytics';
}

protected function getDescription(): ?string
{
    return 'An overview of some analytics.';
}
```
