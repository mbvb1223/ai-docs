# Filament Resources Overview Documentation

## Introduction

Resources are static classes that construct CRUD interfaces for Eloquent models, describing how administrators interact with application data through tables and forms.

## Creating a Resource

Generate a resource with:
```bash
php artisan make:filament-resource Customer
```

This creates a directory structure with:
- `CustomerResource.php` - main resource class
- `Pages/` - full-page Livewire components (Create, Edit, List)
- `Schemas/` - form and infolist definitions
- `Tables/` - table configuration

**Navigation Tip:** Resources require `viewAny()` returning `true` in model policies to appear in navigation menus.

### Resource Variants

**Simple (Modal) Resources:**
```bash
php artisan make:filament-resource Customer --simple
```
Single "Manage" page with modals for create/edit/delete operations.

**Auto-Generated Forms & Tables:**
```bash
php artisan make:filament-resource Customer --generate
```

**Additional Options:**
- `--soft-deletes` - restore, force-delete, filter trashed records
- `--view` - adds View page
- `--model-namespace=Custom\\Path` - specify custom model location
- `--model --migration --factory` - scaffold model and database files

## Record Titles

Set a `$recordTitleAttribute` to identify records:

```php
protected static ?string $recordTitleAttribute = 'name';
```

Supports Eloquent accessors for complex identification needs.

## Resource Forms

Forms are defined in form schema classes or directly in resources:

```php
public static function form(Schema $schema): Schema
{
    return $schema
        ->components([
            TextInput::make('name')->required(),
            TextInput::make('email')->email()->required(),
        ]);
}
```

### Conditional Field Visibility

Hide fields on specific operations:

```php
TextInput::make('password')
    ->password()
    ->required()
    ->hiddenOn(Operation::Edit)
```

Or show only on specific operations:
```php
->visibleOn(Operation::Create)
```

## Resource Tables

Table configuration follows similar structure:

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            TextColumn::make('name'),
            TextColumn::make('email'),
        ])
        ->filters([
            Filter::make('verified')
                ->query(fn (Builder $query): Builder =>
                    $query->whereNotNull('email_verified_at')),
        ])
        ->recordActions([
            EditAction::make(),
        ]);
}
```

## Model Labels

Customize singular and plural labels:

```php
protected static ?string $modelLabel = 'cliente';
protected static ?string $pluralModelLabel = 'clientes';
```

Or use dynamic methods:
```php
public static function getModelLabel(): string
{
    return __('filament/resources/customer.label');
}

public static function getPluralModelLabel(): string
{
    return __('filament/resources/customer.plural_label');
}
```

Disable automatic capitalization:
```php
protected static bool $hasTitleCaseModelLabel = false;
```

## Navigation Configuration

**Custom Label:**
```php
protected static ?string $navigationLabel = 'Mis Clientes';
```

**Icons:**
```php
protected static string | BackedEnum | null $navigationIcon = 'heroicon-o-user-group';
```

**Sorting:**
```php
protected static ?int $navigationSort = 2;
```

**Grouping:**
```php
protected static string | UnitEnum | null $navigationGroup = 'Shop';
```

**Parent Items:**
```php
protected static ?string $navigationParentItem = 'Products';
```

## URL Generation

Generate resource URLs programmatically:

```php
CustomerResource::getUrl();                    // /admin/customers
CustomerResource::getUrl('create');            // /admin/customers/create
CustomerResource::getUrl('edit', ['record' => $customer]);
```

For modals:
```php
CustomerResource::getUrl(parameters: [
    'tableAction' => EditAction::getDefaultName(),
    'tableActionRecord' => $customer,
]);
```

Multi-panel support:
```php
CustomerResource::getUrl(panel: 'marketing');
```

## Customizing Eloquent Queries

Modify queries globally:

```php
public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()->where('is_active', true);
}
```

Remove global scopes:
```php
return parent::getEloquentQuery()->withoutGlobalScopes();
return parent::getEloquentQuery()
    ->withoutGlobalScopes([ActiveScope::class]);
```

## Customizing URL Slug

```php
protected static ?string $slug = 'pending-orders';
```

## Resource Sub-Navigation

Enable navigation between related pages:

```php
public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        ViewCustomer::class,
        EditCustomer::class,
        ManageCustomerAddresses::class,
    ]);
}
```

**Position Options:**
```php
use Filament\Pages\Enums\SubNavigationPosition;

protected static ?SubNavigationPosition $subNavigationPosition =
    SubNavigationPosition::End;
```

Values: `Start`, `End`, or `Top` (renders as tabs).

## Deleting Resource Pages

Remove page files and corresponding entries in `getPages()`:

```php
public static function getPages(): array
{
    return [
        'index' => ListCustomers::route('/'),
        'edit' => EditCustomer::route('/{record}/edit'),
    ];
}
```

## Security & Authorization

Filament observes model policies with these methods:
- `viewAny()` - hide from navigation
- `create()` - control record creation
- `update()` - control record editing
- `view()` - control record viewing
- `delete()` / `deleteAny()` - control deletion
- `forceDelete()` / `forceDeleteAny()` - control force deletion
- `restore()` / `restoreAny()` - control restoration
- `reorder()` - control table reordering

### Skip Authorization

```php
protected static bool $shouldSkipAuthorization = true;
```

### Protect Model Attributes

Remove sensitive attributes from JavaScript:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    unset($data['is_admin']);
    return $data;
}
```
