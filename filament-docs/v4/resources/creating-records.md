# Creating Records - Filament Documentation

## Overview

This documentation covers how to customize the record creation process in Filament, a Laravel admin panel builder.

## Key Customization Methods

### Data Mutation Before Saving

You can modify form data prior to database insertion using the `mutateFormDataBeforeCreate()` method:

```php
protected function mutateFormDataBeforeCreate(array $data): array
{
    $data['user_id'] = auth()->id();
    return $data;
}
```

### Creation Process Control

The `handleRecordCreation()` method allows you to customize how records are instantiated:

```php
protected function handleRecordCreation(array $data): Model
{
    return static::getModel()::create($data);
}
```

### Redirect Configuration

By default, users are redirected to the Edit or View page after creation. Customize this with `getRedirectUrl()`:

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

Panel-wide defaults can be set via configuration: `resourceCreatePageRedirect('index|view|edit')`

### Success Notifications

Customize the success notification through `getCreatedNotificationTitle()` or `getCreatedNotification()`. Return `null` to disable notifications entirely.

## Advanced Features

### Create Another Record

- Disable via `$canCreateAnother = false` property
- Preserve specific form fields using `preserveFormDataWhenCreatingAnother()`

### Lifecycle Hooks

Available hooks include:
- `beforeFill()` / `afterFill()`
- `beforeValidate()` / `afterValidate()`
- `beforeCreate()` / `afterCreate()`

Call `$this->halt()` to interrupt the creation process.

### Multi-Step Wizard

Use the `HasWizard` trait and define steps via `getSteps()`:

```php
use CreateRecord\Concerns\HasWizard;

protected function getSteps(): array
{
    return [
        Step::make('Name')->schema([...]),
        Step::make('Description')->schema([...]),
    ];
}
```

### Record Importing

Add CSV import capability using `ImportAction` with a custom importer class.

### Custom Actions

Add header or form-level actions via `getHeaderActions()` and `getFormActions()` methods.

### Page Content & Views

Override the page schema through `content()` or use a custom Blade view by setting the `$view` property.

## Authorization

Filament respects Laravel model policies. Users need `create()` policy permission to access the Create page.
