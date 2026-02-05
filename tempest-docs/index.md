# Tempest PHP Framework Documentation

Tempest is a PHP development framework designed to minimize friction and maximize productivity. It represents "the sweet spot between the robustness of Symfony and the eloquence of Laravel."

## Core Philosophy

The framework is built on four foundational principles:

1. **Community-Driven**: Started as an educational project, Tempest now has three core maintainers and 50+ contributors, with an active Discord community of ~400 members.

2. **Modern PHP Embracement**: The framework leverages contemporary PHP features including property hooks, attributes, and proxy objects from inception.

3. **Non-Intrusive Design**: Rather than imposing structural requirements, Tempest supports MVC, DDD, hexagonal architecture, and microservices patterns equally well through its "discovery" mechanism.

4. **Innovative Approaches**: Features like console commands modeled after controller actions and decoupled ORM models challenge conventional framework patterns.

## Key Features

- **Discovery system**: Automatically recognizes routing, commands, components, listeners, and migrations without explicit configuration
- **Extensibility**: Framework components can be replaced via container injection
- **View engine**: Uses HTML as its template language with custom directives

## Documentation Structure

### Getting Started
- [Introduction](getting-started/introduction.md)
- [Installation](getting-started/installation.md)

### Essentials
- [Routing](essentials/routing.md)
- [Views](essentials/views.md)
- [Database](essentials/database.md)
- [Console Commands](essentials/console-commands.md)
- [Container](essentials/container.md)
- [Discovery](essentials/discovery.md)
- [Configuration](essentials/configuration.md)
- [Testing](essentials/testing.md)
- [Primitive Utilities](essentials/primitive-utilities.md)

### Features
- [Mapper](features/mapper.md)
- [Asset Bundling](features/asset-bundling.md)
- [Validation](features/validation.md)
- [Authentication](features/authentication.md)
- [File Storage](features/file-storage.md)
- [Cache](features/cache.md)
- [Mail](features/mail.md)
- [Event Bus](features/events.md)
- [Logging](features/logging.md)
- [Command Bus](features/command-bus.md)
- [Localization](features/localization.md)
- [Scheduling](features/scheduling.md)
- [Static Pages](features/static-pages.md)
- [Exception Handling](features/exception-handling.md)
- [Date and Time](features/datetime.md)
- [Processes](features/process.md)
- [OAuth](features/oauth.md)

### Packages
- [Highlight](packages/highlight.md)
- [Console](packages/console.md)

### Internals
- [Framework Lifecycle](internals/lifecycle.md)
- [View Specifications](internals/view-spec.md)

### Extra Topics
- [Roadmap](extra-topics/roadmap.md)
- [Package Development](extra-topics/package-development.md)
- [Standalone Components](extra-topics/standalone-components.md)
- [Deployments](extra-topics/deployments.md)
- [Contributing](extra-topics/contributing.md)

## Source

- **Documentation Source**: https://tempestphp.com/3.x/
- **License**: Content is under [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
