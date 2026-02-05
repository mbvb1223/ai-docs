# Filament Multi-Tenancy Documentation

## Overview

Filament is a Server-Driven UI (SDUI) framework for Laravel that enables developers to define interfaces entirely in PHP using configuration objects rather than traditional templating.

## Key Characteristics

### Technology Stack
Filament builds upon:
- Livewire
- Alpine.js
- Tailwind CSS

### Primary Use Cases
The framework supports creation of:
- Admin panels
- Custom dashboards
- User portals
- CRMs
- Full-featured applications with multiple panels

### Architectural Approach
Server-Driven UI differs from server-rendered approaches by "dynamically generating the UI based on real-time configurations and business logic" rather than relying on static templates defined upfront.

## Package Ecosystem

Filament comprises several modular packages:

| Package | Purpose |
|---------|---------|
| `filament/filament` | Core panel building functionality |
| `filament/tables` | Interactive data table builder |
| `filament/schemas` | Component-based UI configuration system |
| `filament/forms` | Form field inputs with validation |
| `filament/infolists` | Read-only information display components |
| `filament/actions` | Button and modal interaction logic |
| `filament/notifications` | User notification delivery |
| `filament/widgets` | Dashboard statistical components |
| `filament/support` | Shared utilities and base components |

## Extensibility & Plugins

Filament supports community-created plugins distributed via Composer. While the official Filament team maintains several plugins, community-created extensions are managed independently.

## Styling Customization

Filament uses Tailwind CSS as its design system foundation, allowing customization through semantic CSS classes.

## Testing Support

The framework includes testing utilities compatible with both Pest and PHPUnit test suites for verifying functionality and UI components.
