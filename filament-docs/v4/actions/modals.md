# Filament Actions: Modals Documentation

## Overview

Filament actions support modals to gather user confirmation or additional input before executing. The documentation covers confirmation modals, custom schemas, forms, wizards, and extensive customization options.

## Key Features

### Confirmation Modals

Basic confirmation requires the `requiresConfirmation()` method:

```php
Action::make('delete')
    ->action(fn (Post $record) => $record->delete())
    ->requiresConfirmation()
```

### Modal Content Customization

**Text and Labels:**
- `modalHeading()` - Sets the modal title
- `modalDescription()` - Adds descriptive text
- `modalSubmitActionLabel()` - Customizes submit button text

**Schema Rendering:**

Filament allows rendering complete schemas (forms, fields, or infolists) within modals:

```php
Action::make('viewUser')
    ->schema([
        TextInput::make('name'),
        Select::make('position')
            ->options([...])
    ])
```

**Form Data:**

Form field data submits via the `$data` array in the action closure. Use `fillForm()` to pre-populate fields with existing data.

**Wizards:**

Multi-step forms use `steps()` instead of `schema()`:

```php
Action::make('create')
    ->steps([
        Step::make('Name')->schema([...]),
        Step::make('Description')->schema([...])
    ])
```

### Visual Elements

- `modalIcon()` - Adds an icon with optional `modalIconColor()`
- `modalAlignment()` - Controls content alignment (Start/Center)
- `stickyModalHeader()` / `stickyModalFooter()` - Keeps headers/footers visible when scrolling

### Layout Options

- `slideOver()` - Opens as a right-sliding panel instead of centered modal
- `modalWidth()` - Adjusts width using Tailwind scale (ExtraSmall through SevenExtraLarge)
- `disabledForm()` - Makes all form fields read-only

## Advanced Customization

**Custom Content:**
- `modalContent()` - Renders custom Blade views
- `modalContentFooter()` - Adds content below the form
- `extraModalWindowAttributes()` - Adds HTML attributes to the modal element

**Footer Actions:**
- `modalSubmitAction()` / `modalCancelAction()` - Customizes button behavior
- `extraModalFooterActions()` - Adds additional action buttons
- `makeModalSubmitAction()` - Creates extra submit variants with custom arguments

**Modal Behavior:**
- `closeModalByClickingAway(false)` - Prevents dismissal by clicking outside
- `closeModalByEscaping(false)` - Disables escape key closure
- `modalCloseButton(false)` - Hides the close button
- `modalAutofocus(false)` - Prevents auto-focusing first element
- `overlayParentActions()` - Shows child modals over parent modals

## Nested Actions

Child actions can be nested within parent modals. Use `cancelParentActions()` to close parent actions when a child completes. Access parent action data via the `$mountedActions` parameter.

## Form Mounting

The `mountUsing()` method executes code when a modal opens. Remember to call `$form->fill()` if overriding this behavior.

## Conditional Display

Use `modalHidden()` to conditionally show modals, falling back to direct action execution when false.

## Optimization

The `modal()` method tells Filament a modal exists without re-checking, useful when modal configuration involves expensive operations.
