# Filament Documentation: Customizing Styling Overview

## Overview

Filament is a Server-Driven UI (SDUI) framework for Laravel that enables developers to define user interfaces entirely in PHP using structured configuration objects rather than traditional templating.

### Core Architecture

Built on Livewire, Alpine.js, and Tailwind CSS, the framework empowers creation of admin panels, dashboards, and form-based applications without custom JavaScript or frontend code.

## Styling Approach

Filament uses Tailwind CSS as a token-based design system. Rather than direct utility class application, it compiles utilities into semantic CSS, allowing targeted overrides without complete stylesheet rewrites.

### Customization Example

Default button styling uses semantic classes (`.fi-btn`), which developers can override by applying alternative Tailwind utilities in custom CSS.

## Key Distinction: SDUI vs Server-Rendered UI

The documentation clarifies an important difference: while Server-Rendered UI relies on static templates (like Blade views), Server-Driven UI allows servers to dynamically generate interfaces based on real-time configurations and business logic, enabling greater flexibility and reactivity.

## Supported Use Cases

Beyond admin panels, Filament supports:
- Custom dashboards
- User portals
- CRM systems
- Full applications with multiple panels
- Integration with Inertia.js, Livewire, and Blade
- Enhancement of existing Blade views

## Core Packages

Filament comprises foundational packages:

1. **filament/filament** – Panel building (requires all other packages)
2. **filament/tables** – Interactive data tables
3. **filament/schemas** – Component-based UI configuration
4. **filament/forms** – Form inputs with validation
5. **filament/infolists** – Read-only information displays
6. **filament/actions** – Button and modal logic encapsulation
7. **filament/notifications** – User notifications (flash, database, broadcast)
8. **filament/widgets** – Dashboard statistical components
9. **filament/support** – Shared utilities across packages

## Extensibility & Plugins

The framework supports extensive customization through:
- In-application custom components
- Community-maintained Composer packages ("plugins")
- Official Filament team plugins
- Premium community plugins

## Testing & Quality

The framework undergoes unit testing for stability. Users can test applications using Pest or PHPUnit, with particular emphasis recommended when implementing custom functionality.

## Alternatives Mentioned

For different project needs, the documentation suggests:
- **Laravel Nova** – Simpler admin panels
- **Statamic** – Ready-built CMS
- **Flux** – Official Livewire UI kit for Blade-first approaches
