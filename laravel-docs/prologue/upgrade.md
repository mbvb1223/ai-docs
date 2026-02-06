# Laravel 12.x Upgrade Guide

## Overview
Estimated Upgrade Time: **5 Minutes**

This guide covers upgrading from Laravel 11.x to Laravel 12.x. Use [Laravel Shift](https://laravelshift.com/) to automate upgrades.

---

## High Impact Changes

### Updating Dependencies

Update the following in your `composer.json`:

```json
{
  "laravel/framework": "^12.0",
  "phpunit/phpunit": "^11.0",
  "pestphp/pest": "^3.0"
}
```

**Carbon 3 Requirement**: Support for Carbon 2.x has been removed. All Laravel 12 applications now require [Carbon 3.x](https://carbon.nesbot.com/guide/getting-started/migration.html).

### Updating the Laravel Installer

If using the Laravel installer CLI:

```bash
composer global update laravel/installer
```

Or reinstall via `php.new`:

**macOS:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

**Windows (PowerShell as Administrator):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

**Linux:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Or update [Laravel Herd](https://herd.laravel.com) to the latest release.

---

## Medium Impact Changes

### Models and UUIDv7

The `HasUuids` trait now uses UUIDs compatible with version 7 of the UUID spec (ordered UUIDs).

To continue using ordered UUIDv4 strings:

```php
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids;
```

The `HasVersion7Uuids` trait has been removed. Use `HasUuids` instead (now provides the same behavior).

---

## Low Impact Changes

### Carbon 3
Support for Carbon 2.x has been removed.

### Concurrency Result Index Mapping

When invoking `Concurrency::run` with an associative array, results now return with their associated keys:

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

### Container Class Dependency Resolution

The container now respects default values of class properties:

```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// Laravel <= 11.x: $example->date instanceof Carbon;
// Laravel >= 12.x: $example->date === null;
```

### Image Validation Now Excludes SVGs

The `image` validation rule no longer allows SVGs by default:

```php
use Illuminate\Validation\Rules\File;

// Explicitly allow SVGs
'photo' => 'required|image:allow_svg'

// Or using the File rule
'photo' => ['required', File::image(allowSvg: true)],
```

### Local Filesystem Disk Default Root Path

If you don't explicitly define a `local` disk, Laravel now defaults to `storage/app/private` instead of `storage/app`. Define the disk manually to restore previous behavior.

### Multi-Schema Database Inspecting

Schema methods now include results from all schemas by default:

```php
// All tables on all schemas
$tables = Schema::getTables();

// All tables on the 'main' schema
$tables = Schema::getTables(schema: 'main');

// All tables on multiple schemas
$tables = Schema::getTables(schema: ['main', 'blog']);
```

`Schema::getTableListing()` now returns schema-qualified names by default:

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

### Nested Array Request Merging

The `$request->mergeIfMissing()` method now supports "dot" notation for nested arrays:

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

### Database Constructor Signature Changes

**For package maintainers:** Several low-level database classes now require a `Connection` instance:

```php
// Laravel <= 11.x
$grammar = new MySqlGrammar;
$grammar->setConnection($connection);

// Laravel >= 12.x
$grammar = new MySqlGrammar($connection);
```

Deprecated methods:
- `Blueprint::getPrefix()`
- `Grammar::getTablePrefix()` and `setTablePrefix()`
- `Connection::withTablePrefix()`
- `Grammar::setConnection()` (removed)

Retrieve table prefixes directly:

```php
$prefix = $connection->getTablePrefix();
```

### Authentication

The `DatabaseTokenRepository` constructor now expects the `$expires` parameter in **seconds** instead of minutes.

---

## Additional Resources

- Review changes in the [laravel/laravel repository](https://github.com/laravel/laravel)
- Use the [GitHub comparison tool](https://github.com/laravel/laravel/compare/11.x...12.x) to view specific changes
- Consult the [Release Notes](releases.md) for more details

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/upgrade)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
