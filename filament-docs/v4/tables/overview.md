# Filament Tables Overview Documentation

## Introduction

Filament provides a PHP-based API for defining tables with many features while maintaining customizability. The framework uses Eloquent to retrieve row data, and developers define columns through a structured approach.

## Defining Table Columns

Columns are stored as objects within the `$table->columns()` method. Filament includes multiple pre-built column types, and custom column types can be created for specialized data display needs.

Example structure:
```php
use Filament\Tables\Columns\IconColumn;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Table;

public function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('title'),
            TextColumn::make('slug'),
            IconColumn::make('is_featured')
                ->boolean(),
        ]);
}
```

### Making Columns Searchable and Sortable

The `searchable()` method enables filtering rows by column values:
```php
TextColumn::make('title')->searchable()
```

The `sortable()` method adds sort functionality to column headers:
```php
TextColumn::make('title')->sortable()
```

### Displaying Related Data

Using dot notation allows access to relationship attributes:
```php
TextColumn::make('author.name')
```

Filament automatically eager-loads these relationships for performance optimization.

### Adding Columns to Existing Configurations

The `pushColumns()` method appends columns without overriding existing ones:
```php
$table->pushColumns([
    TextColumn::make('created_at')
        ->label('Created')
        ->sortable()
        ->toggleable(isToggledHiddenByDefault: true),
]);
```

## Defining Table Filters

Filters can be defined in the `$table->filters()` method, allowing users to narrow table results through various mechanisms.

Example:
```php
use Filament\Tables\Filters\Filter;
use Filament\Tables\Filters\SelectFilter;

->filters([
    Filter::make('is_featured')
        ->query(fn (Builder $query) => $query->where('is_featured', true)),
    SelectFilter::make('status')
        ->options([
            'draft' => 'Draft',
            'reviewing' => 'Reviewing',
            'published' => 'Published',
        ]),
])
```

Any schema component can be used to build custom filter interfaces, such as date range filters.

## Defining Table Actions

Actions function as buttons for row operations or header commands. Record actions appear at the row end, while header actions support bulk operations.

Example:
```php
->recordActions([
    Action::make('feature')
        ->action(function (Post $record) {
            $record->is_featured = true;
            $record->save();
        })
        ->hidden(fn (Post $record): bool => $record->is_featured),
])
->toolbarActions([
    BulkActionGroup::make([
        DeleteBulkAction::make(),
    ]),
])
```

## Pagination

Tables are paginated by default with options for 5, 10, 25, and 50 records per page.

### Customization Options

```php
// Custom pagination options
$table->paginated([10, 25, 50, 100, 'all']);

// Default page option
$table->defaultPaginationPageOption(25);

// Extreme pagination links
$table->extremePaginationLinks();

// Simple pagination mode
$table->paginationMode(PaginationMode::Simple);

// Cursor pagination
$table->paginationMode(PaginationMode::Cursor);

// Disable pagination
$table->paginated(false);
```

### Preventing Query String Conflicts

Use `queryStringIdentifier()` for multiple tables on one page:
```php
$table->queryStringIdentifier('users');
```

## Record URLs (Clickable Rows)

Make entire rows clickable:
```php
$table->recordUrl(
    fn (Model $record): string => route('posts.edit', ['record' => $record]),
);

// Open in new tab
$table->openRecordUrlInNewTab();
```

## Reordering Records

Enable drag-and-drop reordering:
```php
$table->reorderable('sort');
```

The specified column stores record order. Compatible with packages like `spatie/eloquent-sortable`.

### Reordering Configuration

```php
// Conditional reordering
$table->reorderable('sort', auth()->user()->isAdmin());

// Descending order
$table->reorderable('sort', direction: 'desc');

// Enable pagination during reordering
$table->paginatedWhileReordering();

// Custom trigger action
$table->reorderRecordsTriggerAction(
    fn (Action $action, bool $isReordering) => $action
        ->button()
        ->label($isReordering ? 'Disable reordering' : 'Enable reordering'),
);
```

### Lifecycle Hooks

```php
$table
    ->beforeReordering(function (array $order): void {
        // Runs before database update
    })
    ->afterReordering(function (array $order): void {
        // Runs after database update
    });
```

## Table Header Customization

```php
// Add heading
$table->heading('Clients');

// Add description
$table->description('Manage your clients here.');

// Custom header view
$table->header(view('tables.header', [
    'heading' => 'Clients',
]));
```

## Content Polling

Enable automatic table refresh at specified intervals:
```php
$table->poll('10s');
```

## Deferred Loading

For data-heavy tables, load content asynchronously:
```php
$table->deferLoading();
```

## Laravel Scout Integration

Integrate with Scout's full-text search:
```php
$table->searchUsing(fn (Builder $query, string $search) =>
    $query->whereKey(Post::search($search)->keys())
);
```

At least one searchable column is required for the global search input.

## Styling Table Rows

### Striped Rows

```php
$table->striped();
```

### Custom Row Classes

```php
$table->recordClasses(fn (Post $record) => match ($record->status) {
    'draft' => 'draft-post-table-row',
    'reviewing' => 'reviewing-post-table-row',
    'published' => 'published-post-table-row',
    default => null,
});
```

## Global Settings

Configure defaults for all tables in a service provider's `boot()` method:

```php
use Filament\Tables\Table;

Table::configureUsing(function (Table $table): void {
    $table
        ->reorderableColumns()
        ->filtersLayout(FiltersLayout::AboveContentCollapsible)
        ->paginationPageOptions([10, 25, 50]);
});
```
