# Repeater - Forms - Filament Documentation

## Introduction

The repeater component enables developers to output JSON arrays of repeated form components. Data should be stored in JSON columns with array casting in Eloquent models.

```php
use Filament\Forms\Components\Repeater;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\TextInput;

Repeater::make('members')
    ->schema([
        TextInput::make('name')->required(),
        Select::make('role')
            ->options([
                'member' => 'Member',
                'administrator' => 'Administrator',
                'owner' => 'Owner',
            ])
            ->required(),
    ])
    ->columns(2)
```

## Setting Empty Default Items

Define default empty items using `defaultItems()`. This applies only when loading schemas without data—typically on Create pages in panel resources:

```php
Repeater::make('members')
    ->schema([...])
    ->defaultItems(3)
```

The method accepts functions for dynamic calculation with utility injection support.

## Adding Items

An action button displays below the repeater for adding new items.

### Label Customization

Customize the add button label with `addActionLabel()`:

```php
Repeater::make('members')
    ->schema([...])
    ->addActionLabel('Add member')
```

### Button Alignment

Align the add button using `addActionAlignment()` with `Alignment::Start` or `Alignment::End`:

```php
Repeater::make('members')
    ->schema([...])
    ->addActionAlignment(Alignment::Start)
```

### Disabling Item Addition

Prevent users from adding items:

```php
Repeater::make('members')
    ->schema([...])
    ->addable(false)
```

## Deleting Items

Delete buttons appear on each item by default.

### Preventing Deletion

Disable item deletion functionality:

```php
Repeater::make('members')
    ->schema([...])
    ->deletable(false)
```

## Reordering Items

Drag-and-drop reordering is enabled by default for list manipulation.

### Disabling Reordering

```php
Repeater::make('members')
    ->schema([...])
    ->reorderable(false)
```

### Button-Based Reordering

Enable up/down movement buttons instead of drag-and-drop:

```php
Repeater::make('members')
    ->schema([...])
    ->reorderableWithButtons()
```

### Disabling Drag-and-Drop

```php
Repeater::make('members')
    ->schema([...])
    ->reorderableWithDragAndDrop(false)
```

## Collapsing Items

### Making Collapsible

Hide content in long forms:

```php
Repeater::make('qualifications')
    ->schema([...])
    ->collapsible()
```

### Collapsed by Default

```php
Repeater::make('qualifications')
    ->schema([...])
    ->collapsed()
```

## Cloning Items

Allow item duplication:

```php
Repeater::make('qualifications')
    ->schema([...])
    ->cloneable()
```

## Eloquent Relationship Integration

Use `relationship()` to integrate with `HasMany` relationships:

```php
Repeater::make('qualifications')
    ->relationship()
    ->schema([...])
```

Filament automatically loads and saves relationship data.

### Reordering Related Items

Enable reordering by specifying an order column:

```php
Repeater::make('qualifications')
    ->relationship()
    ->schema([...])
    ->orderColumn('sort')
```

### BelongsToMany Relationships

For `BelongsToMany` relationships, create a pivot model and use it with a `HasMany` relationship on the parent. This example uses an `OrderProduct` pivot model:

```php
use Illuminate\Database\Eloquent\Relations\HasMany;

public function orderProducts(): HasMany
{
    return $this->hasMany(OrderProduct::class);
}
```

The pivot model requires a primary key:

```php
class OrderProduct extends Pivot
{
    public $incrementing = true;
}
```

Then use it in the repeater:

```php
Repeater::make('orderProducts')
    ->relationship()
    ->schema([
        Select::make('product_id')
            ->relationship('product', 'name')
            ->required(),
    ])
```

### Mutating Related Data

**Before Filling:**
```php
Repeater::make('qualifications')
    ->relationship()
    ->schema([...])
    ->mutateRelationshipDataBeforeFillUsing(function (array $data): array {
        $data['user_id'] = auth()->id();
        return $data;
    })
```

**Before Creating:**
```php
->mutateRelationshipDataBeforeCreateUsing(function (array $data): array {
    $data['user_id'] = auth()->id();
    return $data;
})
```

**Before Saving:**
```php
->mutateRelationshipDataBeforeSaveUsing(function (array $data): array {
    $data['user_id'] = auth()->id();
    return $data;
})
```

### Modifying Records After Retrieval

Filter or modify related records post-retrieval:

```php
Repeater::make('startItems')
    ->relationship(
        name: 'items',
        modifyRecordsUsing: fn (Collection $records): Collection =>
            $records->where('group', 'start')
    )
```

## Grid Layout

Organize items into responsive columns:

```php
Repeater::make('qualifications')
    ->schema([...])
    ->grid(2)
```

## Item Labels

Add dynamic labels based on item content:

```php
Repeater::make('members')
    ->schema([
        TextInput::make('name')
            ->required()
            ->live(onBlur: true),
        Select::make('role')
            ->options([...])
            ->required(),
    ])
    ->columns(2)
    ->itemLabel(fn (array $state): ?string => $state['name'] ?? null)
```

Fields used in labels should be `live()` for real-time updates.

## Numbering Items

Add item numbers next to labels:

```php
Repeater::make('members')
    ->schema([...])
    ->itemNumbers()
```

## Simple Repeaters

Create minimal single-field repeaters:

```php
Repeater::make('invitations')
    ->simple(
        TextInput::make('email')
            ->email()
            ->required(),
    )
```

Simple repeaters store data as flat arrays rather than nested objects.

## Accessing Parent Field Values

Use `../` syntax to access fields outside the current repeater item:

- `$get('field_name')` - access sibling field
- `$get('../parent_field')` - access parent-level field
- `$get('../../root_field')` - access root-level field

Example data structure:
```
[
    'client_id' => 1,
    'repeater' => [
        'item1' => ['service_id' => 2]
    ]
]
```

Access `client_id` from inside an item: `$get('../../client_id')`

## Table Repeaters

Present items in table format using `table()`:

```php
use Filament\Forms\Components\Repeater\TableColumn;

Repeater::make('members')
    ->table([
        TableColumn::make('Name'),
        TableColumn::make('Role'),
    ])
    ->schema([
        TextInput::make('name')->required(),
        Select::make('role')->options([...])->required(),
    ])
```

### Table Column Methods

- `hiddenHeaderLabel()` - hide header while maintaining accessibility
- `markAsRequired()` - display red asterisk
- `wrapHeader()` - enable header text wrapping
- `alignment(Alignment::Start|Center|End)` - set alignment
- `width('200px')` - set fixed column width

### Compact Table Repeaters

Make tables more space-efficient:

```php
Repeater::make('members')
    ->table([...])
    ->compact()
    ->schema([...])
```

## Validation

### Minimum and Maximum Items

Validate item counts:

```php
Repeater::make('members')
    ->schema([...])
    ->minItems(2)
    ->maxItems(5)
```

### Distinct State Validation

Ensure uniqueness across items:

```php
Checkbox::make('is_correct')
    ->distinct()
```

For boolean fields, only one item can have `true`. For select/radio/checkbox-list/toggle-buttons, each option appears once maximum.

### Auto-Fixing Indistinct State

Automatically correct duplicates:

```php
Checkbox::make('is_correct')
    ->fixIndistinctState()
```

Boolean fields automatically disable other enabled items when one is selected. Select fields auto-deselect duplicate options.

### Disabling Options in Sibling Items

Disable already-selected options in other repeater items:

```php
Select::make('role')
    ->options([...])
    ->disableOptionsWhenSelectedInSiblingRepeaterItems()
```

Combine with additional conditions using `disableOptionWhen(merge: true)`.

## Customizing Repeater Actions

Customize built-in actions:

```php
use Filament\Actions\Action;

Repeater::make('members')
    ->schema([...])
    ->collapseAllAction(
        fn (Action $action) => $action->label('Collapse all members'),
    )
```

Available action methods:
- `addAction()`
- `cloneAction()`
- `collapseAction()` / `collapseAllAction()`
- `deleteAction()`
- `expandAction()` / `expandAllAction()`
- `moveDownAction()` / `moveUpAction()`
- `reorderAction()`

### Action Confirmation Modals

Add confirmation for destructive actions:

```php
Repeater::make('members')
    ->schema([...])
    ->deleteAction(
        fn (Action $action) => $action->requiresConfirmation(),
    )
```

Note: Collapse, expand, and reorder actions don't support modals.

### Extra Item Actions

Add custom action buttons to repeater items:

```php
use Filament\Actions\Action;
use Filament\Support\Icons\Heroicon;

Repeater::make('members')
    ->schema([
        TextInput::make('email')
            ->label('Email address')
            ->email(),
    ])
    ->extraItemActions([
        Action::make('sendEmail')
            ->icon(Heroicon::Envelope)
            ->action(function (array $arguments, Repeater $component): void {
                $itemData = $component->getItemState($arguments['item']);
                Mail::to($itemData['email'])->send(...);
            }),
    ])
```

- `getItemState()` - get validated item data
- `getRawItemState()` - get unvalidated item data
- `getState()` - get entire repeater raw data
- `state($state)` - set entire repeater data
