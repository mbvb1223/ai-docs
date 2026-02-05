# Deleting Records in Filament

## Overview

Filament provides comprehensive functionality for deleting records in your resources, with built-in support for soft deletes, bulk operations, and authorization policies.

## Handling Soft Deletes

### Creating a Resource with Soft Delete Support

Generate a new resource with soft-delete functionality using the CLI flag:

```bash
php artisan make:filament-resource Customer --soft-deletes
```

### Adding Soft Deletes to Existing Resources

To add soft-delete capabilities to an established resource, update your resource class:

```php
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\ForceDeleteAction;
use Filament\Actions\ForceDeleteBulkAction;
use Filament\Actions\RestoreAction;
use Filament\Actions\RestoreBulkAction;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->filters([
            TrashedFilter::make(),
            // ...
        ])
        ->recordActions([
            DeleteAction::make(),
            ForceDeleteAction::make(),
            RestoreAction::make(),
            // ...
        ])
        ->toolbarActions([
            BulkActionGroup::make([
                DeleteBulkAction::make(),
                ForceDeleteBulkAction::make(),
                RestoreBulkAction::make(),
                // ...
            ]),
        ]);
}

public static function getRecordRouteBindingEloquentQuery(): Builder
{
    return parent::getRecordRouteBindingEloquentQuery()
        ->withoutGlobalScopes([
            SoftDeletingScope::class,
        ]);
}
```

Update your Edit page class to include delete actions:

```php
use Filament\Actions;

protected function getHeaderActions(): array
{
    return [
        Actions\DeleteAction::make(),
        Actions\ForceDeleteAction::make(),
        Actions\RestoreAction::make(),
        // ...
    ];
}
```

## Deleting on the List Page

Add single-record deletion capability to your table:

```php
use Filament\Actions\DeleteAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([
            // ...
        ])
        ->recordActions([
            DeleteAction::make(),
        ]);
}
```

## Authorization

Filament respects Laravel model policies for delete operations:

- **Single deletion**: The `delete()` policy method controls permissions
- **Bulk deletion**: The `deleteAny()` method is used for performance (avoids iterating each record)
- **Force deletion**: `forceDelete()` and `forceDeleteAny()` methods control permissions
- **Restoration**: `restore()` and `restoreAny()` methods control permissions

Use `authorizeIndividualRecords()` on bulk actions to check individual `delete()` policies if needed.
