# Tabs - Schemas Documentation

## Introduction

The Tabs component allows you to organize complex schemas by grouping content into separate, visually distinct sections. This is particularly useful when dealing with lengthy or multifaceted forms.

Basic implementation:

```php
use Filament\Schemas\Components\Tabs;
use Filament\Schemas\Components\Tabs\Tab;

Tabs::make('Tabs')
    ->tabs([
        Tab::make('Tab 1')
            ->schema([
                // ...
            ]),
        Tab::make('Tab 2')
            ->schema([
                // ...
            ]),
    ])
```

## Key Features

### Default Active Tab

Control which tab displays initially using `activeTab()`:

```php
Tabs::make('Tabs')
    ->tabs([...])
    ->activeTab(2)
```

Supports dynamic calculation through utility injection.

### Tab Icons

Add visual indicators using the `icon()` method:

```php
Tab::make('Notifications')
    ->icon(Heroicon::Bell)
    ->schema([...])
```

Position icons before or after labels with `iconPosition()`:

```php
->iconPosition(IconPosition::After)
```

### Tab Badges

Display badge counters:

```php
Tab::make('Notifications')
    ->badge(5)
    ->badgeColor('info')
    ->schema([...])
```

### Grid Customization

Control column layout within tabs:

```php
Tab::make('Tab 1')
    ->schema([...])
    ->columns(3)
```

### Display Variations

**Non-scrollable tabs** (shows dropdown on overflow):
```php
->scrollable(false)
```

**Vertical orientation**:
```php
->vertical()
```

**Remove card styling**:
```php
->contained(false)
```

## Persistence Options

### Session Storage

Store active tab selection in browser storage:

```php
Tabs::make('Tabs')
    ->tabs([...])
    ->persistTab()
    ->id('order-tabs')
```

### URL Query String

Persist selection in URL parameters:

```php
->persistTabInQueryString()
// or with custom key:
->persistTabInQueryString('settings-tab')
```

## Utility Injection

Most methods support dynamic values through function parameters including `$get`, `$record`, `$operation`, `$model`, `$livewire`, and `$component`.

```php
Tab::make('Settings')
    ->visible(fn ($record) => $record?->isAdmin())
    ->schema([...])
```
