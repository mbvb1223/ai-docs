# Editing Records in Filament - Documentation

## Key Customization Methods

The documentation outlines several ways to customize the record editing experience:

**Data Preparation**: You can modify record data before the form loads using `mutateFormDataBeforeFill()`, which allows you to adjust the `$data` array before it populates form fields.

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    $data['user_id'] = auth()->id();
    return $data;
}
```

**Pre-Save Modifications**: The `mutateFormDataBeforeSave()` method lets you transform data before database persistence, useful for adding metadata like "last edited by" information.

```php
protected function mutateFormDataBeforeSave(array $data): array
{
    $data['updated_by'] = auth()->id();
    return $data;
}
```

**Save Process Control**: Override `handleRecordUpdate()` to customize how the Eloquent model receives updates, giving you complete control over the persistence layer.

```php
protected function handleRecordUpdate(Model $record, array $data): Model
{
    $record->update($data);
    return $record;
}
```

## Navigation & User Feedback

**Redirect Behavior**: By default, saving doesn't redirect users. You can customize this via `getRedirectUrl()` or configure default behavior globally through panel settings to redirect to index or view pages.

```php
protected function getRedirectUrl(): string
{
    return $this->getResource()::getUrl('index');
}
```

**Notifications**: Customize success notifications through `getSavedNotificationTitle()` or `getSavedNotification()` methods, or disable them entirely by returning `null`.

## Advanced Features

**Lifecycle Hooks**: Edit pages support multiple hooks including `beforeFill()`, `afterFill()`, `beforeValidate()`, `afterValidate()`, `beforeSave()`, and `afterSave()` for granular control over the editing workflow.

**Partial Saves**: Save specific form sections independently using section actions and `saveFormComponentOnly()`.

**Process Halting**: Call `$this->halt()` to stop the save operation—useful for subscription checks or validation beyond standard form rules.

## Authorization & Extensibility

Access is controlled through Laravel model policies. The `update()` method determines edit access, while `delete()` controls deletion capabilities on the page.
