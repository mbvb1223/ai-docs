# Filament Chart Widgets Documentation

## What is Filament?

Filament is a Server-Driven UI (SDUI) framework for Laravel that enables developers to build user interfaces entirely through PHP configuration objects rather than traditional templating.

### Core Concept

The framework leverages Livewire, Alpine.js, and Tailwind CSS to create full-featured applications like admin panels, dashboards, and form-based systems without requiring custom JavaScript or frontend code development.

### Server-Driven UI vs. Server-Rendered UI

A key distinction exists between these approaches:

- **SDUI**: Allows the server to dynamically generate UI based on real-time configurations and business logic
- **Server-Rendered UI**: Relies on static templates where UI structure is defined upfront

This architecture, used by Meta, Airbnb, and Shopify, centralizes UI control on the server for faster iteration and consistency.

## Chart Widget Creation

Create chart widgets using the artisan command:

```bash
php artisan make:filament-widget BlogPostsChart --chart
```

## Chart Types

Filament supports various chart types powered by Chart.js:

- Line charts
- Bar charts
- Pie charts
- Doughnut charts
- Polar area charts
- Radar charts
- Scatter charts
- Bubble charts

## Basic Implementation

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\ChartWidget;

class BlogPostsChart extends ChartWidget
{
    protected static ?string $heading = 'Blog Posts';

    protected function getData(): array
    {
        return [
            'datasets' => [
                [
                    'label' => 'Blog posts',
                    'data' => [0, 10, 5, 2, 21, 32, 45, 74, 65, 45, 77, 89],
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

## Configuration Options

### Polling

Enable automatic refresh:

```php
protected ?string $pollingInterval = '10s';
```

### Colors

Customize chart colors:

```php
protected function getData(): array
{
    return [
        'datasets' => [
            [
                'label' => 'Posts',
                'data' => [0, 10, 5, 2, 21],
                'backgroundColor' => '#36A2EB',
                'borderColor' => '#9BD0F5',
            ],
        ],
        'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
    ];
}
```

### Chart.js Options

Override default options:

```php
protected function getOptions(): array
{
    return [
        'scales' => [
            'y' => [
                'beginAtZero' => true,
            ],
        ],
    ];
}
```

## Dashboard Integration

Register widgets in your panel provider or dashboard class to display them on the dashboard.
