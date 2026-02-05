# Managing Relationships - Filament Documentation

## Overview

Filament provides multiple approaches for managing Eloquent relationships in admin panels. The appropriate tool depends on relationship type and desired user interface.

## Relationship Management Tools

### Relation Managers
"Interactive tables that allow administrators to list, create, attach, associate, edit, detach, dissociate and delete related records without leaving the resource's Edit or View page."

**Compatible with:** HasMany, HasManyThrough, BelongsToMany, MorphMany, MorphToMany

### Select & Checkbox List
Enables users to select from existing records or create new ones via modal, with automatic pivot table management for many-to-many relationships.

**Compatible with:** BelongsTo, MorphTo, BelongsToMany

### Repeaters
"Standard form components, which can render a repeatable set of fields infinitely" with automatic CRUD operations for related records.

**Compatible with:** HasMany, MorphMany

### Layout Form Components
"All layout form components have a `relationship()` method. When you use this, all fields within that layout are saved to the related model instead of the owner's model."

**Compatible with:** BelongsTo, HasOne, MorphOne

## Creating Relation Managers

### Generation Command
```bash
php artisan make:filament-relation-manager CategoryResource posts title
```

Parameters:
- Resource class name
- Relationship name
- Identifying attribute

### Registration
Register in the resource's `getRelations()` method:

```php
public static function getRelations(): array
{
    return [
        RelationManagers\PostsRelationManager::class,
    ];
}
```

### Read-Only Mode
The View page automatically hides modification actions. Override behavior by implementing `isReadOnly()` method or disabling globally in panel configuration:

```php
->readOnlyRelationManagersOnResourceViewPagesByDefault(false)
```

### Soft Deletes
Use the `--soft-deletes` flag during generation to include restore and force-delete functionality.

## Form and Table Configuration

### Basic Structure
```php
public function form(Schema $schema): Schema
{
    return $schema->components([
        Forms\Components\TextInput::make('title')->required(),
    ]);
}

public function table(Table $table): Table
{
    return $table->columns([
        Tables\Columns\TextColumn::make('title'),
    ]);
}
```

## Pivot Attribute Management

For BelongsToMany and MorphToMany relationships, include pivot attributes in form and table definitions:

```php
// Ensure pivot attributes are listed in withPivot()
Tables\Columns\TextColumn::make('role');
Forms\Components\TextInput::make('role')->required();
```

## Record Operations

### Creating Records
Customize via `CreateAction` in the Actions documentation. For pivot attributes, add them to the form schema.

### Editing Records
Modify pivot attributes using the same form approach as creation.

### Attaching/Detaching
Add `AttachAction`, `DetachAction`, and `DetachBulkAction` for many-to-many relationships:

```php
->headerActions([AttachAction::make()])
->recordActions([DetachAction::make()])
->toolbarActions([
    BulkActionGroup::make([DetachBulkAction::make()])
])
```

**Key methods:**
- `preloadRecordSelect()` - Load options on form initialization
- `multiple()` - Select multiple records
- `recordSelectOptionsQuery()` - Scope available options
- `recordSelectSearchColumns()` - Search across multiple columns

### Associating/Dissociating
Similar to attach/detach but for HasMany/MorphMany relationships. Use `--associate` flag during generation.

### Deleting Records
Enable soft-delete support with `--soft-deletes` flag or manually add:
- `DeleteAction`, `DeleteBulkAction`
- `ForceDeleteAction`, `ForceDeleteBulkAction`
- `RestoreAction`, `RestoreBulkAction`
- `TrashedFilter`

## Advanced Features

### Accessing Owner Records
```php
$this->getOwnerRecord() // In instance methods
// In static methods (form/table):
fn (RelationManager $livewire) => $livewire->getOwnerRecord()
```

### Grouping Managers
```php
RelationGroup::make('Contacts', [
    RelationManagers\IndividualsRelationManager::class,
    RelationManagers\OrganizationsRelationManager::class,
])
```

### Conditional Visibility
```php
public static function canViewForRecord(Model $ownerRecord, string $pageClass): bool
{
    return $ownerRecord->status === Status::Draft;
}
```

### Combined Tabs with Form
Override in Edit/View page class:
```php
public function hasCombinedRelationManagerTabsWithContent(): bool
{
    return true;
}
```

### Query Customization
```php
public function table(Table $table): Table
{
    return $table
        ->modifyQueryUsing(fn (Builder $query) =>
            $query->where('is_active', true)
        );
}
```

### Custom Titles
```php
protected static ?string $title = 'Posts';

// Or dynamically:
public static function getTitle(Model $ownerRecord, string $pageClass): string
{
    return __('relation-managers.posts.title');
}
```

### Record Title Attribute
Set during generation (third argument) or override:
```php
public function table(Table $table): Table
{
    return $table->recordTitle(
        fn (Post $record) => "{$record->title} ({$record->id})"
    );
}
```

## Relation Pages Alternative

Use `ManageRelatedRecords` pages to keep relationship management separate from owner record editing:

```bash
php artisan make:filament-page ManageCustomerAddresses \
    --resource=CustomerResource \
    --type=ManageRelatedRecords
```

Register in `getPages()`:
```php
'addresses' => Pages\ManageCustomerAddresses::route('/{record}/addresses'),
```

Integrate with sub-navigation:
```php
public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        Pages\ManageCustomerAddresses::class,
    ]);
}
```

## Sharing Configuration

Reuse resource form/table definitions in relation managers:

```php
public function form(Schema $schema): Schema
{
    return PostResource::form($schema);
}

public function table(Table $table): Table
{
    return PostResource::table($table);
}
```

Hide specific fields using `hiddenOn()`:
```php
Select::make('post_id')
    ->relationship('post', 'title')
    ->hiddenOn(CommentsRelationManager::class)
```

## Performance Optimization

### Bulk Action Chunking
```php
DetachBulkAction::make()->chunkSelectedRecords(250)
```

### Skip Record Fetching
When individual authorization and model events aren't needed:
```php
DetachBulkAction::make()->fetchSelectedRecords(false)
```

## Lazy Loading

Disabled by default. To prevent lazy loading:
```php
protected static bool $isLazy = false;
```
