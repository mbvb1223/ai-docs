# Deployments

## Overview

This guide covers deploying Tempest applications to production environments.

## Prerequisites

Your production server requires:
- PHP 8.4 or higher
- Composer
- Bun or Node.js (if bundling front-end assets)
- SSH access (recommended over shared hosting)

## Deployment Scripts

Tempest currently lacks a built-in deployment command, though a `tempest ship` command in the future is planned. However, creating deployment automation is straightforward with SSH access.

### Example Deploy Script

The Tempest documentation site uses a simple deployment approach:

```bash
#!/bin/bash

# Source bashrc for SSH connections
. /home/user/.bashrc

# Dependencies
composer install --no-dev
bun install

# Tempest commands
tempest cache:clear --force --internal --all
tempest discovery:generate
tempest migrate:up --force
tempest static:clean --force
bun run build
tempest static:generate --allow-dead-links --verbose=true
```

### Deployment Steps

Production deployments involve:
1. Installing Composer and frontend dependencies
2. Clearing caches and regenerating discovery
3. Running database migrations
4. Cleaning static assets (if applicable)
5. Compiling frontend assets
6. Regenerating static pages (if applicable)

## Initial Server Setup

Before using deployment scripts, perform these one-time configuration steps.

### Environment Configuration

Create a `.env` file in your project root:

```
SIGNING_KEY=…
ENVIRONMENT=production
BASE_URI=https://tempestphp.com
INTERNAL_CACHES=true
DISCOVERY_CACHE=true
PHP_EXECUTABLE_PATH=/usr/bin/php8.4
```

Run `tempest key:generate` to create a signing key.

### Directory Permissions & Scheduling

- Ensure the `.tempest` cache directory is writable by your web server
- Enable the application scheduler for background tasks

## Additional Resources

For missing information or issues, contribute to the [GitHub repository](https://github.com/tempestphp/tempest-framework).
