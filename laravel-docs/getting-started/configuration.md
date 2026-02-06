# Laravel 12.x Configuration Documentation

## Introduction

All configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, allowing you to review available configuration options for:

- Database connection information
- Mail server information
- Core configuration values (application URL, encryption key, etc.)

### The `about` Command

Display an overview of your application's configuration, drivers, and environment:

```bash
php artisan about
```

Filter output by section:

```bash
php artisan about --only=environment
```

Explore specific configuration file values:

```bash
php artisan config:show database
```

---

## Environment Configuration

Use different configuration values based on your environment. Laravel uses [DotEnv](https://github.com/vlucas/phpdotenv) PHP library with `.env` files.

### Environment File Security

**Important:** Never commit your `.env` file to source control. Each developer/server may require different configuration, and it's a security risk.

**Solution:** Use Laravel's built-in environment encryption to safely add encrypted files to source control.

### Additional Environment Files

Laravel checks for `APP_ENV` variable or `--env` CLI argument. If specified, it loads `.env.[APP_ENV]` file; otherwise, defaults to `.env`.

### Environment Variable Types

All `.env` variables are parsed as strings, but reserved values enable wider type returns:

| `.env` Value | `env()` Return Value |
|---|---|
| `true` | `(bool) true` |
| `(true)` | `(bool) true` |
| `false` | `(bool) false` |
| `(false)` | `(bool) false` |
| `empty` | `(string) ''` |
| `(empty)` | `(string) ''` |
| `null` | `(null) null` |
| `(null)` | `(null) null` |

For values with spaces, use double quotes:

```
APP_NAME="My Application"
```

### Retrieving Environment Configuration

Access variables via the `env()` function in configuration files:

```php
'debug' => (bool) env('APP_DEBUG', false),
```

The second parameter is the default value if the variable doesn't exist.

### Determining the Current Environment

```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```

Check if environment matches given values:

```php
if (App::environment('local')) {
    // The environment is local
}

if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

Override environment detection with server-level `APP_ENV` variable.

### Encrypting Environment Files

#### Encryption

Encrypt your `.env` file:

```bash
php artisan env:encrypt
```

This creates `.env.encrypted` with the decryption key displayed in output. Provide your own key:

```bash
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

Key length must match the cipher (default: `AES-256-CBC` requires 32 characters).

For multiple environment files:

```bash
php artisan env:encrypt --env=staging
```

#### Readable Variable Names

Retain visible variable names while encrypting values:

```bash
php artisan env:encrypt --readable
```

Output format:

```
APP_NAME=eyJpdiI6...
APP_ENV=eyJpdiI6...
APP_KEY=eyJpdiI6...
APP_DEBUG=eyJpdiI6...
APP_URL=eyJpdiI6...
```

**Note:** Comments and blank lines aren't included in encrypted output.

#### Decryption

Decrypt environment files:

```bash
php artisan env:decrypt
```

Laravel reads the `LARAVEL_ENV_ENCRYPTION_KEY` environment variable. Provide key directly:

```bash
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

Use custom cipher:

```bash
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

For multiple environment files:

```bash
php artisan env:decrypt --env=staging
```

Overwrite existing files:

```bash
php artisan env:decrypt --force
```

---

## Accessing Configuration Values

Use the `Config` facade or `config()` function with "dot" syntax:

```php
use Illuminate\Support\Facades\Config;

$value = Config::get('app.timezone');
$value = config('app.timezone');

// Retrieve a default value
$value = config('app.timezone', 'Asia/Seoul');
```

### Setting Configuration Values at Runtime

```php
Config::set('app.timezone', 'America/Chicago');
config(['app.timezone' => 'America/Chicago']);
```

### Typed Configuration Retrieval

For static analysis, use typed methods (throws exception on type mismatch):

```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
Config::collection('config-key');
```

---

## Configuration Caching

Cache all configuration files into a single file for performance boost:

```bash
php artisan config:cache
```

Run this during production deployment, **not** during development (configuration changes won't load).

**Important:** Once cached, the `.env` file won't be loaded; the `env()` function only returns external/system-level variables. Only call `env()` from configuration files.

Clear cached configuration:

```bash
php artisan config:clear
```

---

## Configuration Publishing

Most configuration files are already published in the `config` directory. Some files like `cors.php` and `view.php` aren't published by default.

Publish any configuration files:

```bash
php artisan config:publish

# Publish all files
php artisan config:publish --all
```

---

## Debug Mode

Control error information display via the `debug` option in `config/app.php`, which respects the `APP_DEBUG` environment variable in `.env`.

**Development:** Set `APP_DEBUG=true`

**Production:** Set `APP_DEBUG=false` (**CRITICAL** - prevents exposing sensitive configuration to end users)

---

## Maintenance Mode

Place your application in maintenance mode to display a custom view during updates.

### Enable Maintenance Mode

```bash
php artisan down
```

Add `Refresh` header to auto-refresh after N seconds:

```bash
php artisan down --refresh=15
```

Set `Retry-After` header:

```bash
php artisan down --retry=60
```

### Bypassing Maintenance Mode

Generate a bypass secret token:

```bash
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

Access the application via secret URL:

```
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

Let Laravel generate the secret:

```bash
php artisan down --with-secret
```

**Note:** Use alpha-numeric characters and dashes only; avoid special URL characters like `?` or `&`.

### Maintenance Mode on Multiple Servers

**File-based (default):** Execute `php artisan down` on each server.

**Cache-based alternative:** Execute on one server, modify `.env`:

```
APP_MAINTENANCE_DRIVER=cache
APP_MAINTENANCE_STORE=database
```

Select a cache store accessible by all servers for consistent status.

### Pre-Rendering the Maintenance Mode View

Render a view before dependencies load:

```bash
php artisan down --render="errors::503"
```

Customize by editing `resources/views/errors/503.blade.php`.

### Redirecting Maintenance Mode Requests

Redirect all requests to a specific URL:

```bash
php artisan down --redirect=/
```

### Disable Maintenance Mode

```bash
php artisan up
```

### Maintenance Mode and Queues

While in maintenance mode, queued jobs won't be handled; they resume normally after exiting maintenance mode.

### Alternatives to Maintenance Mode

For zero-downtime deployment, consider using [Laravel Cloud](https://cloud.laravel.com).

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/configuration)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
