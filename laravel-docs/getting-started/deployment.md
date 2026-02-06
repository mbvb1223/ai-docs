# Laravel 12.x Deployment Documentation

## Introduction

When deploying a Laravel application to production, there are important steps to ensure optimal efficiency and security. This documentation covers essential deployment practices and configurations.

## Server Requirements

Laravel 12.x requires:

- **PHP >= 8.2**
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

## Server Configuration

### Nginx

Example Nginx configuration (customize as needed):

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

**Critical:** Always direct requests to `public/index.php` to prevent exposing sensitive configuration files.

### FrankenPHP

FrankenPHP is a modern PHP application server written in Go:

```bash
frankenphp php-server -r public/
```

For advanced features (Laravel Octane integration, HTTP/3, compression), consult [FrankenPHP's Laravel documentation](https://frankenphp.dev/docs/laravel/).

### Directory Permissions

Laravel requires write permissions for the web server process owner:
- `bootstrap/cache`
- `storage`

## Optimization

Run this command during deployment to cache all optimizable files:

```bash
php artisan optimize
```

To clear optimization caches:

```bash
php artisan optimize:clear
```

### Caching Configuration

```bash
php artisan config:cache
```

**Important:** Only call the `env()` function in configuration files. Once cached, the `.env` file won't be loaded, and `env()` calls will return `null`.

### Caching Events

```bash
php artisan event:cache
```

### Caching Routes

For large applications with many routes:

```bash
php artisan route:cache
```

This reduces all route registrations into a single cached method call, improving performance.

### Caching Views

```bash
php artisan view:cache
```

Precompiles all Blade views to avoid on-demand compilation, improving request performance.

## Reloading Services

After deploying new code, reload long-running services:

```bash
php artisan reload
```

Reloads:
- Queue workers
- Laravel Reverb
- Laravel Octane

**Note:** Laravel Cloud handles this automatically. For other platforms, configure a process monitor to detect service exits and restart them.

## Debug Mode

**Critical Security Setting:** In `config/app.php`:

```php
'debug' => env('APP_DEBUG', false),
```

**Production requirement:** `APP_DEBUG` must always be `false`. Setting it to `true` in production exposes sensitive configuration values.

## Health Route

Laravel includes a built-in health check at `/up` (customizable):

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

- Returns **200** if application boots successfully
- Returns **500** if exceptions occur

Dispatches `Illuminate\Foundation\Events\DiagnosingHealth` event for custom health checks. Throw exceptions in event listeners to report problems.

## Deploying With Laravel Cloud or Forge

### Laravel Cloud

A fully-managed, auto-scaling deployment platform tuned for Laravel featuring:
- Managed compute
- Databases
- Caches
- Object storage

Visit: [Laravel Cloud](https://cloud.laravel.com)

### Laravel Forge

A VPS server management platform for Laravel with:
- Support for DigitalOcean, Linode, AWS, and more
- Automatic installation of Nginx, MySQL, Redis, Memcached, Beanstalk

Visit: [Laravel Forge](https://forge.laravel.com)

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/deployment)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
