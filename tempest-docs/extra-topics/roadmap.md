# Roadmap

## Overview

The Tempest PHP framework has released its first stable version and invites community contributions. Developers interested in framework development can participate via the [GitHub repository](https://github.com/tempestphp/tempest-framework) or [Discord server](https://discord.gg/pPhpTGUMPQ).

## Experimental Features

Several components are marked as experimental, meaning they may change without triggering a major version bump before Tempest 2.0:

- **View system**: Supports Twig or Blade as alternatives to the default implementation
- **Command bus**: Can be replaced with other command bus solutions
- **Authentication and authorization**: Currently lightweight; third-party implementations encouraged
- **ORM**: Doctrine and other ORMs available as alternatives
- **DateTime component**: Carbon and Psl offered as alternatives
- **Mail component**: Added in version 1.4, kept experimental for edge case refinement
- **Cryptography component**: Experimental to allow API iteration

The framework maintainers emphasize that community feedback is essential for stabilizing these components.

## Upcoming Features

The development priority list includes:

- Dedicated API development support
- HTMX integration with the view system
- Form builder functionality
- Event bus and command bus enhancements (transport, async messaging, event sourcing)
- Queuing and messaging capabilities

Community members can suggest additional features or volunteer to help develop these items through GitHub or Discord channels.
