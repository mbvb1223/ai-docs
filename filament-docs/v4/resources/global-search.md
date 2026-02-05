# Global Search - Filament Documentation

## Overview

Global search enables searching across all resource records from anywhere in the application. To activate this feature, you must set a title attribute on your resource.

## Core Requirements

**Setting Up Global Search:**
A resource requires a `$recordTitleAttribute` property to enable global search functionality. Additionally, the resource needs an Edit or View page for search results to link to a URL.

```php
protected static ?string $recordTitleAttribute = 'title';
```

## Customization Options

### Result Titles

You can override the `getGlobalSearchResultTitle()` method to customize how titles appear, supporting plain text, HTML, or Markdown:

```php
use Illuminate\Contracts\Support\Htmlable;

public static function getGlobalSearchResultTitle(Model $record): string | Htmlable
{
    return $record->name;
}
```

### Multi-Column Search

Search across multiple columns using `getGloballySearchableAttributes()`, with dot notation for relationships:

```php
public static function getGloballySearchableAttributes(): array
{
    return ['title', 'slug', 'author.name', 'category.name'];
}
```

### Result Details

Display additional information below search result titles:

```php
public static function getGlobalSearchResultDetails(Model $record): array
{
    return [
        'Author' => $record->author->name,
        'Category' => $record->category->name,
    ];
}
```

To prevent lazy-loading issues, eager-load relationships:

```php
public static function getGlobalSearchEloquentQuery(): Builder
{
    return parent::getGlobalSearchEloquentQuery()->with(['author', 'category']);
}
```

### Custom URLs

Override default linking behavior:

```php
public static function getGlobalSearchResultUrl(Model $record): string
{
    return UserResource::getUrl('edit', ['record' => $record]);
}
```

### Actions on Results

Add action buttons below search results:

```php
use Filament\Actions\Action;

public static function getGlobalSearchResultActions(Model $record): array
{
    return [
        Action::make('edit')
            ->url(static::getUrl('edit', ['record' => $record])),
    ];
}
```

## Configuration Options

| Setting | Method | Purpose |
|---------|--------|---------|
| Result Limit | `$globalSearchResultsLimit` | Control results per resource (default: 50) |
| Position | `globalSearch()` | Move to sidebar or topbar |
| Sorting | `$globalSearchSort` | Order resources in results |
| Key Bindings | `globalSearchKeyBindings()` | Add keyboard shortcuts |
| Debounce | `globalSearchDebounce()` | Default is 500ms |
| Field Suffix | `globalSearchFieldSuffix()` | Display custom suffix text |
| Term Splitting | `$shouldSplitGlobalSearchTerms` | Disable word-by-word searching |

## Disabling Features

Disable global search while keeping the title attribute:

```php
public function panel(Panel $panel): Panel
{
    return $panel
        ->globalSearch(false);
}
```

Disable search term splitting for performance:

```php
protected static ?bool $shouldSplitGlobalSearchTerms = false;
```

## Advanced Configuration

**Keyboard Shortcuts:**
```php
->globalSearchKeyBindings(['command+k', 'ctrl+k']);
```

**Platform-Specific Suffix:**
```php
->globalSearchFieldSuffix(fn (): ?string => match (Platform::detect()) {
    Platform::Windows, Platform::Linux => 'CTRL+K',
    Platform::Mac => '⌘K',
    default => null,
});
```
