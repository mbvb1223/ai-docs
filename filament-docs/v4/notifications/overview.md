# Filament Notifications Overview - Documentation

## Introduction

Filament provides a `Notification` object that uses a fluent API for construction. As stated in the docs: "Calling the `send()` method on the Notification object will dispatch the notification and display it in your application."

Notifications can be sent from anywhere in your codebase, including JavaScript and Livewire components, leveraging the session to flash messages.

## Core Features

### Title
The primary notification message is displayed as the title. Basic HTML elements are supported, and you can use `Str::markdown()` for safe HTML generation from Markdown content.

### Icons and Colors
Notifications can display icons with optional color customization. Convenience methods like `success()`, `warning()`, `danger()`, and `info()` automatically set appropriate icons and colors for common statuses.

### Background Colors
You can add contextual background colors to notifications using the `color()` method.

### Duration Control
- Default display time: 6 seconds
- Custom duration: set in milliseconds or seconds
- Persistent notifications: remain until manually closed by the user

### Body Text
Additional message content can be provided via the `body()` method, supporting safe HTML and Markdown conversion.

### Actions
Notifications support action buttons that can:
- Open URLs (optionally in new tabs)
- Dispatch Livewire events with optional data parameters
- Use `dispatchSelf()` or `dispatchTo()` for component-specific events
- Close the notification automatically after action

## JavaScript Integration

The `FilamentNotification` and `FilamentNotificationAction` objects are accessible via `window.FilamentNotification` and can be imported from the vendor bundle for use in JavaScript files.

## Advanced Features

### Closing Notifications
Notifications can be closed programmatically by dispatching a `close-notification` browser event with the notification's ID. Custom IDs can be assigned when sending notifications for manual reference.

### Positioning
Notification alignment and vertical positioning are configurable through `Notifications::alignment()` and `Notifications::verticalAlignment()` using `Alignment` and `VerticalAlignment` enums.

## Code Examples

**Basic PHP notification:**
```php
Notification::make()
    ->title('Saved successfully')
    ->success()
    ->send();
```

**JavaScript equivalent:**
```javascript
new FilamentNotification()
    .title('Saved successfully')
    .success()
    .send()
```

**With actions:**
```php
Notification::make()
    ->title('Saved successfully')
    ->success()
    ->actions([
        Action::make('view')->button(),
        Action::make('undo')->color('gray')->close(),
    ])
    ->send();
```
