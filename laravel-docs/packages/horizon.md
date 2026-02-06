# Laravel 12.x Horizon Documentation

## Introduction

Laravel Horizon provides a beautiful dashboard and code-driven configuration for Redis queues. Monitor job throughput, runtime, and failures.

## Installation

```bash
composer require laravel/horizon
php artisan horizon:install
```

## Configuration

Define environments and worker processes in `config/horizon.php`:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

## Dashboard Authorization

Configure in `app/Providers/HorizonServiceProvider.php`:

```php
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return in_array($user->email, [
            '[email protected]',
        ]);
    });
}
```

## Balancing Strategies

### Auto Balancing (Default)

```php
'supervisor-1' => [
    'balance' => 'auto',
    'autoScalingStrategy' => 'time', // or 'size'
    'minProcesses' => 1,
    'maxProcesses' => 10,
],
```

### Simple Balancing

```php
'supervisor-1' => [
    'balance' => 'simple',
    'processes' => 10,
],
```

## Running Horizon

```bash
php artisan horizon
```

### Control Commands

```bash
php artisan horizon:pause
php artisan horizon:continue
php artisan horizon:status
php artisan horizon:terminate
```

## Deploying with Supervisor

Create `/etc/supervisor/conf.d/horizon.conf`:

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

## Tags

### Automatic Tagging

Jobs are auto-tagged with attached Eloquent models.

### Manual Tagging

```php
public function tags(): array
{
    return ['render', 'video:'.$this->video->id];
}
```

## Notifications

```php
Horizon::routeMailNotificationsTo('[email protected]');
Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
```

## Metrics

```php
Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/horizon)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
