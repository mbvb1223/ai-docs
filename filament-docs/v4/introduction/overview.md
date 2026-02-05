# Filament Documentation: Introduction & Overview

## What is Filament?

Filament represents a Server-Driven UI (SDUI) framework designed for Laravel applications. Rather than relying on traditional templating, developers can define entire user interfaces using PHP configuration objects.

### Core Technology Stack

The framework is built upon three key technologies:
- **Livewire** - for server-side reactivity
- **Alpine.js** - for lightweight interactivity
- **Tailwind CSS** - for styling and design tokens

### Key Capabilities

"Filament allows you to define user interfaces entirely in PHP using structured configuration objects" without requiring custom JavaScript or frontend code development. This enables construction of admin panels, dashboards, and form-based applications.

While thousands of developers employ Filament for administrative interfaces, its applications extend far beyond this use case. The framework integrates with existing tech stacks and works particularly well alongside Inertia.js, Livewire, and Blade templating.

## Core Packages

Filament comprises several specialized packages:

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

## Design Philosophy

Filament distinguishes itself from traditional server-rendered approaches. "Server-Driven UI gives the server power to dynamically generate the UI based on real-time configurations and business logic," contrasting with static template-based rendering.

## Plugin Ecosystem

The framework features hundreds of community-created plugins distributed as Composer packages. While the Filament team maintains official plugins, community-created extensions are managed independently. "Plugins not maintained by the Filament team are created by independent authors," requiring users to evaluate code quality and security before installation.

## Customization & Testing

**Styling**: Filament leverages Tailwind CSS utilities compiled into semantic CSS, allowing targeted overrides without modifying component HTML directly.

**Testing**: Core packages include unit tests ensuring stability. Users can write tests compatible with both Pest and PHPUnit frameworks.
