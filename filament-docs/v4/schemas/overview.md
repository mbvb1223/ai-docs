# Filament Schemas Documentation

## Overview

Filament's schemas form the foundation of its Server-Driven UI approach. They allow developers to "build user interfaces declaratively using PHP configuration objects" rather than writing HTML or JavaScript manually.

## Core Concepts

**What Schemas Are:**
Schemas are containers for components that define both configuration and data interactions. They use configuration objects representing UI elements like forms, tables, and lists to control server-side rendering.

**Component Types:**

1. **Form fields** - Accept user input with integrated validation (text inputs, selects, checkboxes)
2. **Infolist entries** - Display read-only key-value information (text, icons, images)
3. **Layout components** - Structure other components (grids, tabs, sections, wizards)
4. **Prime components** - Render static content (text, images, buttons)

## Available Components

### Form Fields
TextInput, Select, Checkbox, Toggle, CheckboxList, Radio, DateTimePicker, FileUpload, RichEditor, MarkdownEditor, Repeater, Builder, TagsInput, Textarea, KeyValue, ColorPicker, ToggleButtons, Slider, CodeEditor, Hidden

### Infolist Entries
TextEntry, IconEntry, ImageEntry, ColorEntry, CodeEntry, KeyValueEntry, RepeatableEntry

### Layout Components
Grid, Flex, Fieldset, Section, Tabs, Wizard, Callout, EmptyStates

## Utility Injection

Methods accept functions as parameters, allowing dynamic configuration. Developers can inject utilities by using specific parameter names:

- **$get** - Retrieve form field values
- **$record** - Access current Eloquent record
- **$operation** - Check if schema is in create/edit/view mode
- **$livewire** - Access Livewire component instance
- **$component** - Access current component instance
- **$request** - Inject Laravel dependencies

## Global Configuration

Use `configureUsing()` in service providers to set default behavior across all instances of a component type, though individual overrides remain possible.
