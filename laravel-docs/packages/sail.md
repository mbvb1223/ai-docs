# Laravel 12.x Sail Documentation

## Introduction

Laravel Sail is a lightweight CLI for Laravel's Docker development environment.

## Installation

```bash
composer require laravel/sail --dev
php artisan sail:install
./vendor/bin/sail up
```

### Shell Alias

```bash
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

## Starting and Stopping

```bash
sail up           # Start containers
sail up -d        # Start in background
sail stop         # Stop containers
```

## Executing Commands

```bash
sail php --version
sail composer require laravel/sanctum
sail artisan migrate
sail artisan queue:work
sail npm run dev
sail node --version
```

## Database Services

### MySQL

- Host: `mysql`
- Port: `3306`

### Redis

- Host: `redis`
- Port: `6379`

### Meilisearch

- Host: `http://meilisearch:7700`
- Web UI: `http://localhost:7700`

## Running Tests

```bash
sail test
sail test --group orders
sail dusk          # Browser tests
```

## Email Preview

Mailpit available at `http://localhost:8025`

```env
MAIL_HOST=mailpit
MAIL_PORT=1025
```

## Container CLI

```bash
sail shell         # Bash shell
sail root-shell    # Root shell
sail tinker        # Laravel Tinker
```

## PHP Versions

Update `compose.yaml`:

```yaml
context: ./vendor/laravel/sail/runtimes/8.4
image: sail-8.4/app
```

```bash
sail build --no-cache
sail up
```

## Sharing Your Site

```bash
sail share
sail share --subdomain=my-app
```

## Xdebug

```env
SAIL_XDEBUG_MODE=develop,debug,coverage
```

```bash
sail debug migrate    # Debug Artisan commands
```

## Customization

```bash
sail artisan sail:publish
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/sail)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
