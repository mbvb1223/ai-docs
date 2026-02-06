# Laravel 12.x Installation Guide

## Meet Laravel

Laravel is a web application framework with expressive, elegant syntax. It provides a structure and starting point for creating web applications, allowing developers to focus on building amazing features while the framework handles the details.

### Why Laravel?

#### A Progressive Framework
Laravel grows with you. Whether you're a beginner with access to extensive documentation, guides, and video tutorials, or a senior developer needing robust tools for dependency injection, unit testing, queues, and real-time events, Laravel scales to your needs.

#### A Scalable Framework
Laravel is incredibly scalable thanks to PHP's scaling-friendly nature and built-in support for fast, distributed cache systems like Redis. Laravel applications have been easily scaled to handle hundreds of millions of requests per month. Platforms like [Laravel Cloud](https://cloud.laravel.com) allow you to run applications at nearly limitless scale.

#### An Agent Ready Framework
Laravel's opinionated conventions and well-defined structure make it ideal for AI-assisted development using tools like Cursor and Claude Code. The framework's expressive syntax, comprehensive documentation, and predictable naming conventions enable AI agents to generate accurate, idiomatic code that follows Laravel patterns reliably.

#### A Community Framework
Laravel combines the best packages in the PHP ecosystem and benefits from thousands of talented developers worldwide who contribute to the framework.

---

## Creating a Laravel Application

### Installing PHP and the Laravel Installer

Before creating your first Laravel application, ensure you have:
- [PHP](https://php.net)
- [Composer](https://getcomposer.org)
- [Laravel installer](https://github.com/laravel/installer)
- [Node and NPM](https://nodejs.org) or [Bun](https://bun.sh/) for frontend asset compilation

#### One-Command Installation

For a complete setup, run one of these commands based on your OS:

**macOS:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

**Windows PowerShell (run as administrator):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

**Linux:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

#### Installing via Composer

If you already have PHP and Composer installed:

```bash
composer global require laravel/installer
```

For a graphical PHP installation experience, check out [Laravel Herd](#installation-using-herd).

### Creating an Application

```bash
laravel new example-app
```

The Laravel installer will prompt you to select:
- Preferred testing framework
- Database
- Starter kit

### Starting Development

```bash
cd example-app
npm install && npm run build
composer run dev
```

Your application will be accessible at [http://localhost:8000](http://localhost:8000).

For a head start, consider using one of the [starter kits](starter-kits.md) which provide backend and frontend authentication scaffolding.

---

## Initial Configuration

All configuration files are stored in the `config` directory. Each option is documented, so review them as needed.

Laravel requires minimal additional configuration. You may wish to review `config/app.php` for options like `url` and `locale`.

### Environment Based Configuration

The `.env` file at your application root contains environment-specific configuration values. Each developer/server may require different settings.

**Important:** Do not commit `.env` to version control. This prevents security risks if your repository is compromised.

For more details, see the [configuration documentation](configuration.md#environment-configuration).

### Databases and Migrations

By default, Laravel uses SQLite. A `database/database.sqlite` file is created during setup with necessary migrations.

#### Using MySQL or PostgreSQL

Update your `.env` file's `DB_*` variables:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Then run migrations:

```bash
php artisan migrate
```

For macOS/Windows local development of MySQL, PostgreSQL, or Redis, consider [Herd Pro](https://herd.laravel.com/#plans) or [DBngin](https://dbngin.com/).

### Directory Configuration

Laravel should always be served from the root of the web directory configured for your web server. Never serve from a subdirectory, as this could expose sensitive files.

---

## Installation Using Herd

[Laravel Herd](https://herd.laravel.com) is a blazing fast, native Laravel and PHP development environment for macOS and Windows. It includes everything needed to get started: PHP, Nginx, and command line tools for php, composer, laravel, expose, node, npm, and nvm.

[Herd Pro](https://herd.laravel.com/#plans) adds powerful features:
- Create and manage local MySQL, Postgres, and Redis databases
- Local mail viewing
- Log monitoring

### Herd on macOS

1. Download the installer from [Herd website](https://herd.laravel.com)
2. The installer automatically downloads the latest PHP and configures Nginx to run in the background
3. Herd uses [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) to support "parked" directories
4. By default, Herd creates a parked directory at `~/Herd`

#### Creating a New Application

```bash
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

Manage parked directories and PHP settings via Herd's UI from the system tray menu.

Learn more at [Herd documentation](https://herd.laravel.com/docs).

### Herd on Windows

1. Download the installer from [Herd website](https://herd.laravel.com/windows)
2. Start Herd to complete the onboarding process
3. Access Herd UI via left-click on system tray icon
4. Right-click for quick menu access

#### Creating a New Application

Open PowerShell and run:

```powershell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

Herd creates a parked directory at `%USERPROFILE%\Herd` where applications are automatically served on the `.test` domain.

Learn more at [Herd documentation for Windows](https://herd.laravel.com/docs/windows).

---

## IDE Support

Recommended editors:
- **VS Code / Cursor** with official [Laravel VS Code Extension](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel)
  - Syntax highlighting
  - Snippets
  - Artisan command integration
  - Smart autocompletion for Eloquent models, routes, middleware, assets, config, and Inertia.js

- **PhpStorm** - JetBrains IDE with extensive Laravel support
  - Built-in Laravel framework support
  - Blade templates
  - Smart autocompletion
  - Powerful code generation

- **Firebase Studio** - Cloud-based development
  - Zero setup required
  - Build Laravel directly in your browser from any device

---

## Laravel and AI

### Laravel Boost

[Laravel Boost](https://github.com/laravel/boost) bridges the gap between AI coding agents and Laravel applications. It provides:

- Laravel-specific context, tools, and guidelines
- AI agents with 15+ specialized tools:
  - Know which packages you're using
  - Query your database
  - Search Laravel documentation
  - Read browser logs
  - Generate tests
  - Execute code via Tinker

- Over 17,000 pieces of vectorized Laravel ecosystem documentation (version-specific)
- Laravel-maintained AI guidelines for:
  - Following framework conventions
  - Writing appropriate tests
  - Avoiding common pitfalls

### Installing Laravel Boost

**Requirements:** Laravel 10, 11, or 12 with PHP 8.1+

```bash
composer require laravel/boost --dev
```

Run the interactive installer:

```bash
php artisan boost:install
```

The installer auto-detects your IDE and AI agents, allowing you to opt into relevant features.

#### Adding Custom AI Guidelines

Add `.blade.php` or `.md` files to `.ai/guidelines/*` directory. These are automatically included with Laravel Boost's guidelines when running `boost:install`.

---

## Next Steps

Become familiar with Laravel by reading:
- [Request Lifecycle](../architecture/lifecycle.md)
- [Configuration](configuration.md)
- [Directory Structure](structure.md)
- [Frontend](frontend.md)
- [Service Container](../architecture/container.md)
- [Facades](../architecture/facades.md)

### Laravel the Full Stack Framework

Use Laravel to route requests and render frontend via [Blade templates](../basics/blade.md) or single-page application hybrid technology like [Inertia](https://inertiajs.com).

Recommended documentation:
- [Frontend development](frontend.md)
- [Routing](../basics/routing.md)
- [Views](../basics/views.md)
- [Eloquent ORM](../eloquent/eloquent.md)

Consider community packages:
- [Livewire](https://livewire.laravel.com)
- [Inertia](https://inertiajs.com)

Learn to compile CSS and JavaScript using [Vite](../basics/vite.md).

Check out official [starter kits](starter-kits.md) for a head start.

### Laravel the API Backend

Use Laravel as an API backend for JavaScript SPAs or mobile applications (e.g., [Next.js](https://nextjs.org)).

Recommended documentation:
- [Routing](../basics/routing.md)
- [Laravel Sanctum](../packages/sanctum.md)
- [Eloquent ORM](../eloquent/eloquent.md)

Leverage powerful services:
- [Queues](../digging-deeper/queues.md)
- [Email](../digging-deeper/mail.md)
- [Notifications](../digging-deeper/notifications.md)

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
