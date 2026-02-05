# Filament Select Form Component Documentation

## Overview

The Select component enables users to choose from predefined options. It supports both native HTML5 selects and customizable JavaScript-based selects with advanced features like searching, relationships, and dynamic options.

## Basic Usage

```php
use Filament\Forms\Components\Select;

Select::make('status')
    ->options([
        'draft' => 'Draft',
        'reviewing' => 'Reviewing',
        'published' => 'Published',
    ])
```

Options can be static arrays or dynamically generated through closures with utility injection support.

## JavaScript Select Mode

By default, Filament renders the native HTML5 select. Switch to a more feature-rich JavaScript implementation:

```php
Select::make('status')
    ->options([...])
    ->native(false)
```

## Search Functionality

### Basic Searchable Select

Enable search input for easier option access:

```php
Select::make('author_id')
    ->label('Author')
    ->options(User::query()->pluck('name', 'id'))
    ->searchable()
```

### Custom Search Results

For database-driven searches, use `getSearchResultsUsing()` and `getOptionLabelUsing()`:

```php
Select::make('author_id')
    ->searchable()
    ->getSearchResultsUsing(fn (string $search): array => User::query()
        ->where('name', 'like', "%{$search}%")
        ->limit(50)
        ->pluck('name', 'id')
        ->all())
    ->getOptionLabelUsing(fn ($value): ?string => User::find($value)?->name)
```

### Search Configuration

```php
Select::make('author_id')
    ->searchable()
    ->loadingMessage('Loading authors...')
    ->noSearchResultsMessage('No authors found.')
    ->searchPrompt('Search by name or email')
    ->searchingMessage('Searching...')
    ->searchDebounce(500)  // milliseconds
```

## Multi-Select

Enable multiple selections:

```php
Select::make('technologies')
    ->multiple()
    ->options([
        'tailwind' => 'Tailwind CSS',
        'alpine' => 'Alpine.js',
        'laravel' => 'Laravel',
    ])
```

Store multiple selections as JSON using Eloquent array casts:

```php
protected function casts(): array
{
    return [
        'technologies' => 'array',
    ];
}
```

### Reordering Multiple Selections

```php
Select::make('technologies')
    ->multiple()
    ->reorderable()
    ->options([...])
```

### Multi-Select Label Retrieval

Use `getOptionLabelsUsing()` for multiple selected values:

```php
Select::make('technologies')
    ->multiple()
    ->searchable()
    ->getOptionLabelsUsing(fn (array $values): array => Technology::query()
        ->whereIn('id', $values)
        ->pluck('name', 'id')
        ->all())
```

## Option Grouping

Organize options under categorical labels:

```php
Select::make('status')
    ->searchable()
    ->options([
        'In Process' => [
            'draft' => 'Draft',
            'reviewing' => 'Reviewing',
        ],
        'Reviewed' => [
            'published' => 'Published',
            'rejected' => 'Rejected',
        ],
    ])
```

## Eloquent Relationship Integration

### BelongsTo Relationship

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
```

### BelongsToMany Relationship

```php
Select::make('technologies')
    ->multiple()
    ->relationship(titleAttribute: 'name')
```

### Search Across Multiple Columns

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
    ->searchable(['name', 'email'])
```

### Preload Options

Load all options on page initialization instead of on search:

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
    ->searchable()
    ->preload()
```

### Exclude Current Record

```php
Select::make('parent_id')
    ->relationship(name: 'parent', titleAttribute: 'name', ignoreRecord: true)
```

### Customize Relationship Query

```php
Select::make('author_id')
    ->relationship(
        name: 'author',
        titleAttribute: 'name',
        modifyQueryUsing: fn (Builder $query) => $query->withTrashed(),
    )
```

### Custom Option Labels

Transform Eloquent models into descriptive labels:

```php
Select::make('author_id')
    ->relationship(
        name: 'author',
        modifyQueryUsing: fn (Builder $query) => $query
            ->orderBy('first_name')
            ->orderBy('last_name'),
    )
    ->getOptionLabelFromRecordUsing(fn (Model $record) =>
        "{$record->first_name} {$record->last_name}"
    )
    ->searchable(['first_name', 'last_name'])
```

### Pivot Data Storage

Save additional data to pivot tables in BelongsToMany relationships:

```php
Select::make('primaryTechnologies')
    ->relationship(name: 'technologies', titleAttribute: 'name')
    ->multiple()
    ->pivotData(['is_primary' => true])
```

## Creating Options in Modals

### Create New Option

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
    ->createOptionForm([
        Forms\Components\TextInput::make('name')->required(),
        Forms\Components\TextInput::make('email')->required()->email(),
    ])
```

### Customize Creation Process

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
    ->createOptionForm([...])
    ->createOptionUsing(function (array $data): int {
        return auth()->user()->team->members()->create($data)->getKey();
    })
```

### Edit Selected Option

```php
Select::make('author_id')
    ->relationship(name: 'author', titleAttribute: 'name')
    ->editOptionForm([
        Forms\Components\TextInput::make('name')->required(),
        Forms\Components\TextInput::make('email')->required()->email(),
    ])
```

### Customize Update Process

```php
Select::make('author_id')
    ->editOptionForm([...])
    ->updateOptionUsing(function (array $data, Schema $schema) {
        $schema->getRecord()?->update($data);
    })
```

## MorphTo Relationships

Handle polymorphic relationships with type selection:

```php
use Filament\Forms\Components\MorphToSelect;

MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->titleAttribute('name'),
        MorphToSelect\Type::make(Post::class)
            ->titleAttribute('title'),
    ])
```

### Custom Type Labels

```php
MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->getOptionLabelFromRecordUsing(fn (Product $record): string =>
                "{$record->name} - {$record->slug}"
            ),
    ])
```

### Customize Type Query

```php
MorphToSelect::make('commentable')
    ->types([
        MorphToSelect\Type::make(Product::class)
            ->titleAttribute('name')
            ->modifyOptionsQueryUsing(fn (Builder $query) =>
                $query->whereBelongsTo($this->team)
            ),
    ])
```

### Toggle Button Type Selector

```php
MorphToSelect::make('commentable')
    ->typeSelectToggleButtons()
    ->types([...])
```

## HTML in Option Labels

Enable HTML rendering in option labels:

```php
Select::make('technology')
    ->options([
        'tailwind' => '<span class="text-blue-500">Tailwind</span>',
        'alpine' => '<span class="text-green-500">Alpine</span>',
    ])
    ->searchable()
    ->allowHtml()
```

## Option Label Wrapping

Control how overflowing labels display in JavaScript selects:

```php
Select::make('truncate')
    ->wrapOptionLabels(false)  // Truncate instead of wrap
```

## Placeholder Management

Prevent placeholder selection:

```php
Select::make('status')
    ->options([...])
    ->default('draft')
    ->selectablePlaceholder(false)
```

## Disable Specific Options

```php
Select::make('status')
    ->options([...])
    ->disableOptionWhen(fn (string $value): bool => $value === 'published')
```

## Affixes

Add text before and after the input:

```php
Select::make('domain')
    ->prefix('https://')
    ->suffix('.com')
```

### Icon Affixes

```php
use Filament\Support\Icons\Heroicon;

Select::make('domain')
    ->suffixIcon(Heroicon::GlobeAlt)
    ->suffixIconColor('success')
```

## Utility Injection

Most methods accept closures with injectable parameters including:

- `$component` - Field instance
- `$get` - Value retrieval function
- `$livewire` - Livewire component
- `$model` - Eloquent model FQN
- `$operation` - Current operation (create/edit/view)
- `$record` - Current Eloquent record
- `$state` - Field current value
- `$search` - Search input (searchable selects)
