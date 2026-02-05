# Optimizing Local Development - Filament Documentation

## Overview

This guide provides optional performance optimization tips for running Filament applications in local development environments.

## Enabling OPcache

OPcache improves PHP performance by storing precompiled script bytecode in shared memory, eliminating the need to load and parse scripts on each request.

### Checking OPcache Status

To verify if OPcache is enabled, run:

```bash
php -r "echo 'opcache.enable => ' . ini_get('opcache.enable') . PHP_EOL;"
```

You should see `opcache.enable => 1`. If not, add this to your `php.ini`:

```ini
opcache.enable=1
```

To locate your `php.ini` file: `php --ini`

### Configuring OPcache Settings

Adjust these parameters in `php.ini` if experiencing slow response times:

```ini
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

- **opcache.memory_consumption**: Memory in megabytes for storing precompiled code (start at 128, adjust as needed)
- **opcache.max_accelerated_files**: Maximum PHP files to cache (10000 is a reasonable starting point)

## Exclude Project Folder from Antivirus Scanning

Realtime antivirus scanning can significantly slow development, particularly on Windows. Consider excluding your project folder from realtime scanning in tools like Microsoft Defender or similar security software.

**Note**: Only exclude folders from scanning if you trust the project.

## Disabling Debugging Tools

Debugging tools improve development workflow but impact performance when not actively used.

### Laravel Herd View Debugging

Herd's view debugging tool shows rendered views and passed data. To disable:

1. Open Herd > Dumps
2. Click Settings
3. Uncheck "Views" option

### Laravel Debugbar

Disable by adding to `.env`:

```env
DEBUGBAR_ENABLED=false
```

Alternatively, disable specific collectors you're not using (see [Debugbar documentation](https://github.com/barryvdh/laravel-debugbar)).

### Xdebug

Xdebug significantly impacts performance. Disable in `php.ini`:

```ini
xdebug.mode=off
```

## Caching Blade Icons

Caching Blade icons improves performance in view-heavy applications. Run:

```bash
php artisan icons:cache
```

Re-run this command when installing new Blade icon packages.
