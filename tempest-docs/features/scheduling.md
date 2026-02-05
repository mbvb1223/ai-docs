# Scheduling

## Overview

Tempest provides a modern and convenient way of scheduling tasks, which can be any class method, even existing console commands. The framework uses the `#[Tempest\Console\Schedule]` attribute to mark schedulable methods, with automatic discovery handling registration.

## Using the Scheduler

To execute scheduled tasks on a server, implement a single cron entry that runs the `schedule:run` command:

```
0 * * * * user /path/to/tempest schedule:run
```

This command evaluates which tasks should execute at the current time.

## Defining Schedules

Methods decorated with the `#[Schedule]` attribute are automatically discovered and executed by the scheduler.

### Basic Example

```php
use Tempest\Console\Schedule;
use Tempest\Console\Scheduler\Every;

final readonly class ScheduledTasks
{
    #[Schedule(Every::HOUR)]
    public function updateSlackChannels(): void
    {
        // …
    }
}
```

### Using the Every Enumeration

The `Every` enumeration covers common scheduling scenarios (hourly, daily, weekly, etc.).

### Fine-Grained Control with Interval

For more precise timing, use an `Interval` object:

```php
use Tempest\Console\Schedule;
use Tempest\Console\Scheduler\Interval;

#[Schedule(new Interval(hours: 2, minutes: 30))]
public function updateSlackChannels(): void
{
    // …
}
```

### Combining with Console Commands

Scheduled tasks need not be console commands, but they can be both—useful when you want manual execution capability alongside automatic scheduling:

```php
use Tempest\Console\ConsoleCommand;
use Tempest\Console\Schedule;

#[Schedule(Every::HOUR)]
#[ConsoleCommand('slack:update-channels')]
public function updateSlackChannels(): void
{
    // …
}
```
