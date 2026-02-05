# Filament Tables - Filters Overview Documentation

## Introduction

Filters in Filament allow developers to define constraints on table data, enabling users to scope records to find specific information. They're implemented through the `$table->filters()` method.

Basic filter creation uses the static `make()` method with a unique name and a `query()` callback that applies the filter's scope:

```php
use Filament\Tables\Filters\Filter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public function table(Table $table): Table
{
    return $table
        ->filters([
            Filter::make('is_featured')
                ->query(fn (Builder $query): Builder => $query->where('is_featured', true))
        ]);
}
```

## Available Filter Types

- **Default checkbox filter**: Renders a checkbox component by default
- **Toggle filter**: Replaces checkbox with a toggle button
- **Select filter**: Allows users to choose from options
- **Ternary filter**: Offers three states (true/false/blank)
- **Trashed filter**: Pre-built ternary filter for soft-deletable records
- **Query builder**: Advanced UI for complex filter combinations
- **Custom filters**: Flexible construction with various form fields

## Customization Options

### Labels
Customize filter labels with the `label()` method, supporting static values or dynamic functions:

```php
Filter::make('is_featured')
    ->label('Featured')
```

### Form Field Customization
Replace the default checkbox with other form components or modify built-in fields using `modifyFormFieldUsing()`:

```php
Filter::make('is_featured')
    ->toggle()
```

### Default Application
Enable filters by default with the `default()` method.

### Session Persistence
Preserve filter states across sessions using `persistFiltersInSession()`.

## Filter Behavior

### Live vs. Deferred Filters
By default, filters are deferred and require an "Apply" button. Make them live with `deferFilters(false)`.

### Record Deselection
Records are deselected when filters change by default. Disable this using `deselectAllRecordsWhenFiltered(false)`.

### Base Query Modification
For modifications beyond scoped `where()` clauses, use the `baseQuery()` method to modify the query directly.

### Record Resolution Exclusion
Exclude specific filters when resolving records using `excludeWhenResolvingRecord()`, useful for filters that modify global scopes rather than restrict access.

## Utility Injection

Filter configuration methods support dependency injection with specific parameter names:

- `$filter`: The current filter instance
- `$livewire`: The Livewire component instance
- `$table`: The current table configuration
- Standard Laravel container dependencies (e.g., `Request`)

Parameters can be combined in any order within function closures.
