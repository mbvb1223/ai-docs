# Filament Forms Overview Documentation

## Introduction

Filament's forms package enables developers to build dynamic forms within Laravel applications. The package integrates with other Filament tools like panel resources, action modals, and table filters.

## Core Form Components

Filament provides numerous field types within the `Filament\Form\Components` namespace:

- Text input, Select, Checkbox, Toggle
- Checkbox list, Radio, Date-time picker
- File upload, Rich editor, Markdown editor
- Repeater, Builder, Tags input
- Textarea, Key-value, Color picker
- Toggle buttons, Slider, Code editor
- Hidden fields and custom fields

Fields are instantiated using the `make()` method with a unique identifier, typically matching Eloquent model attributes.

## Field Configuration Methods

### Validation

Fields support Laravel validation through fluent methods like `required()` and `maxLength()`, enabling both frontend and backend validation.

### Labels and Visibility

- **Labels**: Customize with `label()` or hide visually using `hiddenLabel()`
- **Hiding fields**: Use `hidden()` or `visible()` methods
- **JavaScript-based hiding**: Use `hiddenJs()` or `visibleJs()` for client-side logic
- **Operation-based control**: `disabledOn()`, `hiddenOn()`, `visibleOn()` methods

### Defaults and States

- `default()` method sets initial values (used on Create pages)
- `disabled()` prevents editing; `saved()` allows disabled fields to persist
- `inlineLabel()` displays labels beside fields rather than above

## Advanced Field Features

### Content Slots

Fields support multiple content slots for adding extra elements:

- Label slots: `aboveLabel()`, `beforeLabel()`, `afterLabel()`, `belowLabel()`
- Content slots: `aboveContent()`, `beforeContent()`, `afterContent()`, `belowContent()`
- Error message slots: `aboveErrorMessage()`, `belowErrorMessage()`

Slots accept static text, schema components, actions, or mixed arrays of content.

### HTML Attributes

- `extraAttributes()`: Adds attributes to outer element
- `extraInputAttributes()`: Targets input/select elements
- `extraFieldWrapperAttributes()`: Customizes field wrapper styling

### Fused Groups

The `FusedGroup` component visually combines multiple compatible fields (text input, select, date-time picker, color picker) into cohesive units with optional grid layouts.

## Utility Injection

Functions in field methods can receive injected utilities:

- `$component`: Current field instance
- `$get`: Retrieve form data values
- `$livewire`: Livewire component
- `$model`: Eloquent model FQN
- `$operation`: Current operation (create/edit/view)
- `$state`: Current field value
- `$record`: Associated Eloquent record

This enables dynamic field behavior based on form state without network requests.
