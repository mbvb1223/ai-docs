# Logging

## Overview

Tempest provides logging built on Monolog that follows PSR-3 and the RFC 5424 specification. The system supports eight standard log levels and can route messages to multiple destinations simultaneously, including files, Slack, and system logs.

## Writing Logs

Inject the `Logger` interface into any class to begin logging. By default, messages are written to daily rotating files in `.tempest/logs`.

```php
use Tempest\Log\Logger;

final readonly class UserService
{
    public function __construct(
        private Logger $logger,
    ) {}
}
```

### Available Log Levels

The framework supports eight RFC 5424 severity levels:

- `emergency()` — System unusable
- `alert()` — Immediate action required
- `critical()` — Important, unexpected error
- `error()` — Runtime error to monitor
- `warning()` — Exceptional occurrence
- `notice()` — Uncommon event
- `info()` — Miscellaneous event
- `debug()` — Detailed debugging information

## Providing Context

All log methods accept an optional context array containing additional data formatted as JSON:

```php
$logger->error('Order processing failed', context: [
    'user_id' => $order->userId,
    'order_id' => $order->id,
    'total_amount' => $order->total,
]);
```

## Configuration

Default configuration uses daily rotation, retaining up to 31 files:

```php
use Tempest\Log\Config\DailyLogConfig;

return new DailyLogConfig(
    path: Tempest\internal_storage_path('logs', 'tempest.log'),
    maxFiles: Tempest\env('LOG_MAX_FILES', default: 31)
);
```

### Minimum Log Levels

Configure channels to ignore messages below a specified severity:

```php
new DailyLogChannel(
    path: Tempest\internal_storage_path('logs', 'tempest.log'),
    minimumLogLevel: LogLevel::DEBUG,
),
```

### Multiple Loggers

Create separate tagged configurations for different application areas:

```php
// src/Orders/logging.config.php
return new DailyLogConfig(
    path: Tempest\internal_storage_path('logs', 'orders.log'),
    tag: Logging::ORDERS,
);

// src/Orders/ProcessOrder.php
public function __construct(
    #[Tag(Logging::ORDERS)]
    private Logger $logger,
) {}
```

## Available Channels

Tempest provides five log channel types:

- **AppendLogChannel** — Single file, no rotation
- **DailyLogChannel** — New file daily with auto-cleanup
- **WeeklyLogChannel** — New file weekly with auto-cleanup
- **SlackLogChannel** — Send to Slack via webhook
- **SysLogChannel** — Write to system log

Each has a corresponding config class: `SimpleLogConfig`, `DailyLogConfig`, `WeeklyLogConfig`, `SlackLogConfig`, and `SysLogConfig`.

## Debugging Functions

Global helper functions for development:

- `ll()` — Write to debug log (silent)
- `lw()` or `dump()` — Log and display values
- `ld()` or `dd()` — Log, display, and halt execution
- `le()` — Log and emit `ItemsDebugged` event

## Debug Log Management

Monitor real-time debug logs with:

```bash
./tempest tail:debug
```

Use `--no-clear` flag to preserve previous entries.

### Configure Debug Output

Create `config/debug.config.php`:

```php
use Tempest\Debug\DebugConfig;

return new DebugConfig(
    logPath: Tempest\internal_storage_path('logs', 'debug.log')
);
```
