# Laravel 12.x Telescope Documentation

## Introduction

Laravel Telescope is a debugging assistant for local Laravel development providing insight into requests, exceptions, logs, queries, jobs, mail, and more.

## Installation

```bash
composer require laravel/telescope
php artisan telescope:install
php artisan migrate
```

Access dashboard via `/telescope`.

### Local-Only Installation

```bash
composer require laravel/telescope --dev
```

Register conditionally in `AppServiceProvider`:

```php
public function register(): void
{
    if ($this->app->environment('local')) {
        $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

## Configuration

### Data Pruning

```php
Schedule::command('telescope:prune')->daily();
Schedule::command('telescope:prune --hours=48')->daily();
```

### Dashboard Authorization

```php
protected function gate(): void
{
    Gate::define('viewTelescope', function (User $user) {
        return in_array($user->email, [
            '[email protected]',
        ]);
    });
}
```

## Filtering

```php
Telescope::filter(function (IncomingEntry $entry) {
    if ($this->app->environment('local')) {
        return true;
    }

    return $entry->isReportableException() ||
        $entry->isFailedJob() ||
        $entry->isScheduledTask() ||
        $entry->isSlowQuery();
});
```

## Tagging

```php
Telescope::tag(function (IncomingEntry $entry) {
    return $entry->type === EntryType::REQUEST
        ? ['status:'.$entry->content['response_status']]
        : [];
});
```

## Available Watchers

| Watcher | Purpose |
|---------|---------|
| Batch | Queued batch information |
| Cache | Cache operations |
| Command | Artisan commands |
| Dump | Variable dumps |
| Event | Dispatched events |
| Exception | Exceptions with stack traces |
| Gate | Authorization checks |
| HTTP Client | Outgoing HTTP requests |
| Job | Queued jobs |
| Log | Application logs |
| Mail | Sent emails |
| Model | Eloquent events |
| Notification | Sent notifications |
| Query | Database queries |
| Redis | Redis commands |
| Request | Incoming requests |
| Schedule | Scheduled tasks |
| View | Rendered views |

### Watcher Configuration

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => true,
        'slow' => 50, // milliseconds
    ],
    Watchers\RequestWatcher::class => [
        'enabled' => true,
        'size_limit' => 64, // kilobytes
    ],
],
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/telescope)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
