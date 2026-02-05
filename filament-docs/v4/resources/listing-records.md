# Listing Records - Filament Documentation

## Using Tabs to Filter Records

Filament allows you to add tabs above tables for filtering records based on predefined conditions. Each tab can scope the Eloquent query differently.

### Basic Implementation

Register tabs by adding a `getTabs()` method to the List page class:

```php
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

public function getTabs(): array
{
    return [
        'all' => Tab::make(),
        'active' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
        'inactive' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', false)),
    ];
}
```

### Customizing Tab Labels

Override default label generation by passing text to the `make()` method:

```php
public function getTabs(): array
{
    return [
        'all' => Tab::make('All customers'),
        'active' => Tab::make('Active customers')
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true)),
    ];
}
```

### Adding Icons to Tabs

```php
Tab::make()
    ->icon('heroicon-m-user-group')
    ->iconPosition(IconPosition::After)
```

### Adding Badges to Tabs

Display count information or other data:

```php
Tab::make()
    ->badge(Customer::query()->where('active', true)->count())
    ->badgeColor('success')
```

### Extra HTML Attributes

```php
Tab::make()
    ->extraAttributes(['data-cy' => 'statement-confirmed-tab'])
```

### Setting Default Active Tab

```php
public function getDefaultActiveTab(): string | int | null
{
    return 'active';
}
```

### Excluding Tab Query When Resolving Records

When record state changes after display, prevent the tab query from being applied during record resolution:

```php
public function getTabs(): array
{
    return [
        'active' => Tab::make()
            ->modifyQueryUsing(fn (Builder $query) => $query->where('active', true))
            ->excludeQueryWhenResolvingRecord(),
    ];
}
```

**Important Note:** "Do not use `excludeQueryWhenResolvingRecord()` on tabs that enforce authorization rules."

## Authorization

Filament respects registered model policies. Access to the List page requires the `viewAny()` policy method to return `true`. The `reorder()` method controls record reordering permissions.

## Customizing Table Eloquent Query

Make query modifications specific to the List page:

```php
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

public static function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) => $query->withoutGlobalScopes());
}
```

## Custom Page Content

Override the page schema by defining a `content()` method:

```php
use Filament\Schemas\Components\EmbeddedTable;
use Filament\Schemas\Components\RenderHook;
use Filament\Schemas\Schema;

public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->getTabsContentComponent(),
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_BEFORE),
            EmbeddedTable::make(),
            RenderHook::make(PanelsRenderHook::RESOURCE_PAGES_LIST_RECORDS_TABLE_AFTER),
        ]);
}
```

### Using Custom Blade Views

Override the static `$view` property on the page class to point to a custom view location and implement fully customized rendering as needed.
