# Filament v4 Documentation

Filament is a Server-Driven UI (SDUI) framework for Laravel that enables developers to build user interfaces entirely through PHP configuration objects rather than traditional templating.

## Requirements

- PHP 8.2+
- Laravel v11.28+
- Tailwind CSS v4.1+

## Core Technology Stack

- **Livewire** - for server-side reactivity
- **Alpine.js** - for lightweight interactivity
- **Tailwind CSS** - for styling and design tokens

## Documentation Structure

### Introduction
- [Overview](introduction/overview.md) - What is Filament
- [Installation](introduction/installation.md) - Getting started
- [AI-Assisted Development](introduction/ai-assisted-development.md)
- [Optimizing Local Development](introduction/optimizing-local-development.md)

### Getting Started
- [Index](getting-started/index.md) - Core concepts and first steps

### Resources
- [Overview](resources/overview.md) - CRUD interface classes
- [Listing Records](resources/listing-records.md)
- [Creating Records](resources/creating-records.md)
- [Editing Records](resources/editing-records.md)
- [Viewing Records](resources/viewing-records.md)
- [Deleting Records](resources/deleting-records.md)
- [Managing Relationships](resources/managing-relationships.md)
- [Global Search](resources/global-search.md)

### Tables
- [Overview](tables/overview.md) - Data table builder
- [Columns](tables/columns/)
  - [Text Column](tables/columns/text.md)
- [Filters](tables/filters/)
  - [Overview](tables/filters/overview.md)

### Schemas
- [Overview](schemas/overview.md) - Component-based UI
- [Sections](schemas/sections.md)
- [Tabs](schemas/tabs.md)

### Forms
- [Overview](forms/overview.md) - Form components
- [Text Input](forms/text-input.md)
- [Select](forms/select.md)
- [File Upload](forms/file-upload.md)
- [Repeater](forms/repeater.md)
- [Validation](forms/validation.md)

### Infolists
- [Overview](infolists/overview.md) - Read-only data display

### Actions
- [Overview](actions/overview.md) - Button and modal actions
- [Modals](actions/modals.md)

### Notifications
- [Overview](notifications/overview.md) - User notifications

### Widgets
- [Overview](widgets/overview.md) - Dashboard widgets
- [Stats Overview](widgets/stats-overview.md)
- [Chart Widgets](widgets/chart-widgets.md)

### Panel Configuration
- [Index](panel-configuration/index.md) - Admin panel setup

### Navigation
- [Overview](navigation/overview.md) - Sidebar and navigation

### Users
- [Overview](users/overview.md) - Authentication and authorization
- [Multi-tenancy](users/multi-tenancy.md)

### Customizing Styling
- [Overview](customizing-styling/overview.md) - Tailwind CSS customization

### Testing
- [Overview](testing/overview.md) - Testing Filament applications

### Plugins
- [Getting Started](plugins/getting-started.md) - Building plugins

## Core Packages

| Package | Purpose |
|---------|---------|
| `filament/filament` | Core panel building functionality |
| `filament/tables` | Interactive data table creation |
| `filament/schemas` | Component-based UI configuration |
| `filament/forms` | Form field components with validation |
| `filament/infolists` | Read-only information display lists |
| `filament/actions` | Button and modal action encapsulation |
| `filament/notifications` | User notification delivery |
| `filament/widgets` | Dashboard widget components |
| `filament/support` | Shared utilities and components |

## Quick Start

```bash
# Install Filament
composer require filament/filament:"^4.0"
php artisan filament:install --panels

# Create admin user
php artisan make:filament-user
```

Access your panel at `/admin` in your browser.

## License

Filament is open-source software licensed under the MIT license.

---

*Documentation crawled from [https://filamentphp.com/docs/4.x/](https://filamentphp.com/docs/4.x/)*
