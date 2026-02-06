# Laravel 12.x Logging Documentation

## Introduction

Laravel provides robust logging services based on "channels" using the Monolog library.

## Configuration

Configuration is in `config/logging.php`. Default uses the `stack` channel.

### Available Channel Drivers

| Name | Description |
|------|-------------|
| `custom` | Calls a factory to create a channel |
| `daily` | Rotating daily log files |
| `errorlog` | PHP error log |
| `monolog` | Any Monolog handler |
| `papertrail` | SyslogUdp handler |
| `single` | Single file logger |
| `slack` | Slack webhook handler |
| `stack` | Multi-channel aggregator |
| `syslog` | System log handler |

## Building Log Stacks

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'level' => env('LOG_LEVEL', 'critical'),
    ],
],
```

### Log Levels

emergency, alert, critical, error, warning, notice, info, debug

## Writing Log Messages

```php
use Illuminate\Support\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

### Example Usage

```php
Log::info('Showing the user profile for user: {id}', ['id' => $id]);
```

### Adding Context

```php
Log::withContext([
    'request-id' => $requestId
]);

Log::shareContext([
    'request-id' => $requestId
]);
```

### Writing to Specific Channels

```php
Log::channel('slack')->info('Something happened!');
Log::stack(['single', 'slack'])->info('Something happened!');
```

### On-Demand Channels

```php
Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```

## Monolog Channel Customization

### Customizing Channels

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
],
```

### Creating Custom Channels

```php
'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

## Tailing Logs with Pail

```bash
composer require --dev laravel/pail
php artisan pail
```

### Filtering

```bash
php artisan pail --filter="QueryException"
php artisan pail --message="User created"
php artisan pail --level=error
php artisan pail --user=1
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/logging)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
