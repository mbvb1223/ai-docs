# Laravel 12.x Task Scheduling Documentation

## Introduction

Laravel's command scheduler offers a fresh approach to managing scheduled tasks. Define your command schedule fluently in the `routes/console.php` file.

Only a single cron entry is needed:
```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

## Defining Schedules

### Basic Scheduling with Closures

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

### Scheduling Artisan Commands

```php
use App\Console\Commands\SendEmailsCommand;

Schedule::command('emails:send Taylor --force')->daily();
Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

### Scheduling Queued Jobs

```php
use App\Jobs\Heartbeat;

Schedule::job(new Heartbeat)->everyFiveMinutes();
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

### Scheduling Shell Commands

```php
Schedule::exec('node /home/forge/script.js')->daily();
```

## Schedule Frequency Options

### Basic Frequencies

| Method | Description |
|--------|-------------|
| `->everySecond()` | Run every second |
| `->everyMinute()` | Run every minute |
| `->everyFiveMinutes()` | Run every five minutes |
| `->everyTenMinutes()` | Run every ten minutes |
| `->everyFifteenMinutes()` | Run every fifteen minutes |
| `->everyThirtyMinutes()` | Run every thirty minutes |
| `->hourly()` | Run every hour |
| `->hourlyAt(17)` | Run every hour at 17 minutes past |
| `->daily()` | Run every day at midnight |
| `->dailyAt('13:00')` | Run every day at 13:00 |
| `->twiceDaily(1, 13)` | Run daily at 1:00 & 13:00 |
| `->weekly()` | Run every Sunday at 00:00 |
| `->weeklyOn(1, '8:00')` | Run every Monday at 8:00 |
| `->monthly()` | Run on the first day of every month |
| `->monthlyOn(4, '15:00')` | Run monthly on the 4th at 15:00 |
| `->quarterly()` | Run on the first day of every quarter |
| `->yearly()` | Run on the first day of every year |
| `->cron('* * * * *')` | Custom cron schedule |

### Day Constraints

```php
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

### Constraint Methods

| Method | Description |
|--------|-------------|
| `->weekdays()` | Limit to weekdays |
| `->weekends()` | Limit to weekends |
| `->sundays()` through `->saturdays()` | Limit to specific day |
| `->days([0, 3])` | Limit to specific days |
| `->between('7:00', '22:00')` | Limit to time range |
| `->unlessBetween('23:00', '4:00')` | Exclude time range |
| `->when(Closure)` | Conditional execution |
| `->skip(Closure)` | Skip based on condition |
| `->environments('staging', 'production')` | Limit to environments |

## Timezones

```php
Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00');
```

## Preventing Task Overlaps

```php
Schedule::command('emails:send')->withoutOverlapping();
Schedule::command('emails:send')->withoutOverlapping(10);
```

## Running Tasks on One Server

```php
Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

### Naming Single Server Jobs

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

## Background Tasks

```php
Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

## Maintenance Mode

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

## Schedule Groups

```php
Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

## Running the Scheduler

### Server Setup

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

### Sub-Minute Scheduled Tasks

```php
Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

### Running Locally

```bash
php artisan schedule:work
```

## Task Output

```php
Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);

Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);

Schedule::command('report:generate')
    ->daily()
    ->emailOutputTo('[email protected]');

Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('[email protected]');
```

## Task Hooks

### Before and After Hooks

```php
Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // Task is about to execute...
    })
    ->after(function () {
        // Task has executed...
    });
```

### Success and Failure Hooks

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // Task succeeded...
    })
    ->onFailure(function () {
        // Task failed...
    });
```

### Pinging URLs

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url)
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

## Events

- `ScheduledTaskStarting`
- `ScheduledTaskFinished`
- `ScheduledBackgroundTaskFinished`
- `ScheduledTaskSkipped`
- `ScheduledTaskFailed`

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/scheduling)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
