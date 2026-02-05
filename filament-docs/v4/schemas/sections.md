# Filament Sections Documentation

## Overview

Sections are layout components in Filament that allow you to organize form fields into grouped areas, each with optional headings and descriptions.

## Basic Section Creation

```php
use Filament\Schemas\Components\Section;

Section::make('Rate limiting')
    ->description('Prevent abuse by limiting the number of requests per period')
    ->schema([
        // Fields go here
    ])
```

You can also create headerless sections that function as simple cards:

```php
Section::make()
    ->schema([
        // Fields go here
    ])
```

## Key Features

### Icons in Headers
Add visual indicators using the `icon()` method with Heroicon support.

```php
Section::make('Rate limiting')
    ->icon('heroicon-o-clock')
    ->schema([...])
```

### Aside Layout
Use `->aside()` to position the heading and description on the left with content on the right.

```php
Section::make('Settings')
    ->aside()
    ->schema([...])
```

### Collapsible Sections
Make sections expandable/collapsible with `->collapsible()` and set them collapsed by default using `->collapsed()`.

```php
Section::make('Advanced options')
    ->collapsible()
    ->collapsed()
    ->schema([...])
```

### Persist Collapsed State
Enable `->persistCollapsed()` to remember collapse preferences in local storage across page refreshes.

```php
Section::make('Advanced options')
    ->collapsible()
    ->persistCollapsed()
    ->schema([...])
```

### Styling Options
- **Compact**: Use `->compact()` for nested sections
- **Secondary**: Apply `->secondary()` for less contrasting backgrounds

```php
Section::make('Details')
    ->compact()
    ->secondary()
    ->schema([...])
```

### Header and Footer Actions
Insert actions or components using `->afterHeader([])` or `->footer([])` methods.

```php
Section::make('User details')
    ->afterHeader([
        Action::make('edit')->button(),
    ])
    ->footer([
        Action::make('save')->button(),
    ])
    ->schema([...])
```

### Grid Layout
Apply `->columns(2)` to create multi-column layouts within sections.

```php
Section::make('Contact information')
    ->columns(2)
    ->schema([
        TextInput::make('phone'),
        TextInput::make('email'),
    ])
```

## Dynamic Values

Most methods accept functions for dynamic calculation, supporting utility injection including `$get`, `$record`, `$operation`, and more.

```php
Section::make('Admin settings')
    ->visible(fn () => auth()->user()->isAdmin())
    ->schema([...])
```
