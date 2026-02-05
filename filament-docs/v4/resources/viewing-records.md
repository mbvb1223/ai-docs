# Viewing Records - Filament Documentation

## Overview

This documentation section covers how to create and manage View pages in Filament resources, allowing users to display record information in read-only formats.

## Creating Resources with View Pages

### Basic Setup

To generate a new resource with a View page:

```bash
php artisan make:filament-resource User --view
```

### Using Infolists

By default, View pages display disabled forms. For a cleaner presentation, implement an `infolist()` method:

```php
use Filament\Infolists;
use Filament\Schemas\Schema;

public static function infolist(Schema $schema): Schema
{
    return $schema
        ->components([
            Infolists\Components\TextEntry::make('name'),
            Infolists\Components\TextEntry::make('email'),
            Infolists\Components\TextEntry::make('notes')
                ->columnSpanFull(),
        ]);
}
```

## Adding View Pages to Existing Resources

Create a new page using the command:

```bash
php artisan make:filament-page ViewUser --resource=UserResource --type=ViewRecord
```

Register it in `getPages()`:

```php
public static function getPages(): array
{
    return [
        'index' => Pages\ListUsers::route('/'),
        'create' => Pages\CreateUser::route('/create'),
        'view' => Pages\ViewUser::route('/{record}'),
        'edit' => Pages\EditUser::route('/{record}/edit'),
    ];
}
```

## Modal Viewing

For simpler resources, display records in modals by adding a `ViewAction`:

```php
use Filament\Actions\ViewAction;
use Filament\Tables\Table;

public static function table(Table $table): Table
{
    return $table
        ->columns([/* ... */])
        ->recordActions([
            ViewAction::make(),
        ]);
}
```

## Data Mutation

Customize record data before form population using `mutateFormDataBeforeFill()`:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();
    return $data;
}
```

## Lifecycle Hooks

Execute code at specific page lifecycle points:

```php
protected function beforeFill(): void
{
    // Runs before form field population
}

protected function afterFill(): void
{
    // Runs after form field population
}
```

## Authorization

Filament respects Laravel's model policies. The `view()` policy method controls access to View pages.

## Multiple View Pages

Create additional View pages for complex resources using the same command as above. Register each in `getPages()` with unique route names.

## Relation Manager Customization

Specify which relation managers appear on specific View pages:

```php
protected function getAllRelationManagers(): array
{
    return [
        RelationManagers\OrdersRelationManager::class,
        RelationManagers\SubscriptionsRelationManager::class,
    ];
}
```

## Custom Page Content

Override the `content()` method to customize page structure:

```php
public function content(Schema $schema): Schema
{
    return $schema
        ->components([
            $this->hasInfolist()
                ? $this->getInfolistContentComponent()
                : $this->getFormContentComponent(),
            $this->getRelationManagersContentComponent(),
        ]);
}
```

### Custom Blade Views

Replace the default view by setting:

```php
protected string $view = 'filament.resources.users.pages.view-user';
```

Then create the custom view file at `resources/views/filament/resources/users/pages/view-user.blade.php`.
