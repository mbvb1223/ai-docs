# Installation

## Prerequisites

The framework requires PHP 8.5+ and Composer. Optional dependencies include Bun or Node for frontend asset bundling. A complete development environment like ServBay, Herd, or Valet is recommended, though PHP's built-in server works adequately.

## Creating a Tempest Application

Initialize a new project using the `tempest/app` starter template:

```bash
composer create-project tempest/app my-app
cd my-app
```

Access the application via `https://my-app.test` in a dedicated development environment, or use PHP's built-in server:

```bash
php tempest serve
```

### Scaffolding Front-End Assets

Optionally bootstrap a frontend setup with Vite and Tailwind CSS:

```bash
php tempest install vite --tailwind
```

This generates `main.entrypoint.ts` and `main.entrypoint.css`, automatically discovered by Tempest. Serve assets using the `<x-vite-tags />` component in templates.

Run the development server:

```bash
npm run dev
```

## Tempest as a Package

Install the framework into existing projects:

```bash
composer require tempest/framework
```

Access the console via `./vendor/bin/tempest`. Optionally scaffold entry points:

```bash
./vendor/bin/tempest install framework
```

The installer prompts for optional installation of `public/index.php`, `tempest`, `.env.example`, and `.env` files.

## Project Structure

One of its core features is that it will scan all project and package code for you, and will automatically discover any files the framework needs to know about.

Tempest imposes no mandatory file structure. The discovery system differentiates between controller methods and console commands through code inspection rather than naming conventions or configuration.

Both traditional and modular organizational patterns work identically without special configuration.

## About Discovery

Discovery operates by analyzing project code to determine file and method purposes. In production, Tempest caches the discovery process, avoiding any performance overhead.

The system identifies controller methods through route attributes and console commands via `#[ConsoleCommand]` annotations, enabling flexible project organization.

See the [Discovery documentation](../essentials/discovery.md) for additional details.
