# Filament Navigation Documentation

## Overview

Filament automatically registers navigation items for resources, custom pages, and clusters. These classes contain static properties and methods to configure navigation behavior.

## Key Navigation Features

### Customizing Labels
```php
protected static ?string $navigationLabel = 'Custom Navigation Label';
// or
public static function getNavigationLabel(): string
{
    return 'Custom Navigation Label';
}
```

### Icons & Active States
```php
use Filament\Support\Icons\Heroicon;

protected static string | BackedEnum | null $navigationIcon = Heroicon::OutlinedDocumentText;
protected static string | BackedEnum | null $activeNavigationIcon = Heroicon::OutlinedDocumentText;
```

### Sorting Items
```php
protected static ?int $navigationSort = 3;
```
Lower values appear first in ascending order.

### Navigation Badges
```php
public static function getNavigationBadge(): ?string
{
    return static::getModel()::count();
}

public static function getNavigationBadgeColor(): ?string
{
    return static::getModel()::count() > 10 ? 'warning' : 'primary';
}

protected static ?string $navigationBadgeTooltip = 'The number of users';
```

## Grouping Navigation Items

### Basic Grouping
```php
protected static string | UnitEnum | null $navigationGroup = 'Settings';
```

### Nested Groups (Parent Items)
```php
protected static ?string $navigationParentItem = 'Notifications';
protected static string | UnitEnum | null $navigationGroup = 'Settings';
```

### Customizing Groups in Configuration
```php
use Filament\Navigation\NavigationGroup;

->navigationGroups([
    NavigationGroup::make()
         ->label('Shop')
         ->icon(Heroicon::OutlinedShoppingCart),
    NavigationGroup::make()
        ->label('Settings')
        ->icon(Heroicon::OutlinedCog6Tooth)
        ->collapsed(),
])
```

### Group Collapsibility
```php
->collapsibleNavigationGroups(false); // Disable all collapsible groups
```

### Enum-based Groups
```php
enum NavigationGroup implements HasLabel, HasIcon
{
    case Shop;
    case Blog;
    case Settings;

    public function getLabel(): string { /*...*/ }
    public function getIcon(): string | BackedEnum | Htmlable | null { /*...*/ }
}
```

## Sidebar Configuration

### Collapsible Sidebar
```php
->sidebarCollapsibleOnDesktop();
->sidebarFullyCollapsibleOnDesktop();
```

### Sidebar Width
```php
->sidebarWidth('40rem');
->collapsedSidebarWidth('9rem');
```

## Custom Navigation Items
```php
use Filament\Navigation\NavigationItem;

->navigationItems([
    NavigationItem::make('Analytics')
        ->url('https://example.com', shouldOpenInNewTab: true)
        ->icon(Heroicon::OutlinedPresentationChartLine)
        ->group('Reports')
        ->sort(3),
])
```

### Conditional Visibility
```php
NavigationItem::make('Analytics')
    ->visible(fn(): bool => auth()->user()->can('view-analytics'))
    ->hidden(fn(): bool => ! auth()->user()->can('view-analytics'))
```

## Disabling Navigation Items

```php
protected static bool $shouldRegisterNavigation = false;
// or
public static function shouldRegisterNavigation(): bool
{
    return false;
}
```

## Navigation Modes

### Top Navigation
```php
->topNavigation();
```

### Disable Navigation
```php
->navigation(false);
```

### Disable Topbar
```php
->topbar(false);
```

## Advanced Customization

### Custom Navigation Builder
```php
use Filament\Navigation\NavigationBuilder;

->navigation(function (NavigationBuilder $builder): NavigationBuilder {
    return $builder->items([
        NavigationItem::make('Dashboard')->url(Dashboard::getUrl()),
        ...UserResource::getNavigationItems(),
    ]);
})
```

### Custom Components
```php
->sidebarLivewireComponent(CustomSidebar::class)
->topbarLivewireComponent(CustomTopbar::class)
```

## Additional Features

### Breadcrumbs
```php
->breadcrumbs(false); // Disable breadcrumbs
```

### Refreshing Navigation
```php
// From Livewire component
$this->dispatch('refresh-sidebar');

// From JavaScript
window.dispatchEvent(new CustomEvent('refresh-sidebar'));
```
