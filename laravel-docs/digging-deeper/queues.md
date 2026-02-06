# Laravel 12.x Queues Documentation

## Introduction

Laravel queues allow you to defer time-consuming tasks to be processed in the background, enabling your application to respond quickly to web requests. Laravel provides a unified queueing API across various backends including Amazon SQS, Redis, databases, and Beanstalkd.

### Connections vs. Queues

**Connections** define connections to backend queue services (Redis, SQS, etc.), while **Queues** are different stacks of queued jobs within a connection.

```php
// Sent to default queue
ProcessPodcast::dispatch();

// Sent to "emails" queue
ProcessPodcast::dispatch()->onQueue('emails');
```

## Driver Prerequisites

### Database Driver

```bash
php artisan make:queue-table
php artisan migrate
```

### Other Drivers

Required Composer packages:
- **Amazon SQS**: `aws/aws-sdk-php ~3.0`
- **Beanstalkd**: `pda/pheanstalk ~5.0`
- **Redis**: `predis/predis ~2.0` or phpredis extension

## Creating Jobs

### Generating Job Classes

```bash
php artisan make:job ProcessPodcast
```

### Class Structure

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Podcast $podcast,
    ) {}

    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }
}
```

### Unique Jobs

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    public $uniqueFor = 3600; // Seconds

    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```

### Encrypted Jobs

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

## Job Middleware

```php
<?php

namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

class RateLimited
{
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
            ->block(0)->allow(1)->every(5)
            ->then(function () use ($job, $next) {
                $next($job);
            }, function () use ($job) {
                $job->release(5);
            });
    }
}
```

Attach middleware to a job:

```php
public function middleware(): array
{
    return [new RateLimited];
}
```

### Preventing Job Overlaps

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

## Dispatching Jobs

### Basic Dispatch

```php
ProcessPodcast::dispatch($podcast);

// Conditional dispatch
ProcessPodcast::dispatchIf($accountActive, $podcast);
ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

### Delayed Dispatching

```php
ProcessPodcast::dispatch($podcast)
    ->delay(now()->plus(minutes: 10));
```

### Synchronous Dispatching

```php
ProcessPodcast::dispatchSync($podcast);
```

### Job Chaining

```php
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

### Customizing Queue and Connection

```php
ProcessPodcast::dispatch($podcast)->onQueue('processing');
ProcessPodcast::dispatch($podcast)->onConnection('sqs');
ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->onQueue('processing');
```

### Max Attempts and Timeout

```php
class ProcessPodcast implements ShouldQueue
{
    public $tries = 5;
    public $timeout = 120;
    public $failOnTimeout = true;
}
```

Via command line:

```bash
php artisan queue:work --tries=3 --timeout=30
```

## Job Batching

### Create Batches Table

```bash
php artisan make:queue-batches-table
php artisan migrate
```

### Defining Batchable Jobs

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            return;
        }
        // Import portion of CSV
    }
}
```

### Dispatching Batches

```php
use Illuminate\Support\Facades\Bus;

$batch = Bus::batch([
    new ImportCsv(1, 100),
    new ImportCsv(101, 200),
])->then(function (Batch $batch) {
    // Batch completed
})->catch(function (Batch $batch, Throwable $e) {
    // Job failed
})->finally(function (Batch $batch) {
    // Batch completed or failed
})->dispatch();
```

## Running the Queue Worker

```bash
php artisan queue:work
```

### The `queue:work` Command

```bash
php artisan queue:work --queue=processing
php artisan queue:work --queue=high,default
php artisan queue:work --timeout=30
php artisan queue:work --tries=3
php artisan queue:work --once
php artisan queue:work --max-jobs=1000
php artisan queue:work --max-time=3600
```

## Supervisor Configuration

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/laravel/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
numprocs=8
redirect_stderr=true
stdout_logfile=/var/log/laravel-worker.log
stopwaitsecs=3600
```

## Dealing With Failed Jobs

### Cleaning Up After Failed Jobs

```php
public function failed(Throwable $exception): void
{
    // Send user notification, etc.
}
```

### Retrying Failed Jobs

```bash
php artisan queue:failed
php artisan queue:retry {id}
php artisan queue:retry --all
```

### Pruning Failed Jobs

```bash
php artisan queue:prune-failed --hours=48
```

## Testing

### Faking Jobs

```php
use Illuminate\Support\Facades\Queue;

Queue::fake();

ProcessPodcast::dispatch($podcast);

Queue::assertPushed(ProcessPodcast::class);
```

## Job Events

```php
Queue::before(function (JobProcessing $event) {
    // Before job execution
});

Queue::after(function (JobProcessed $event) {
    // After successful execution
});

Queue::failed(function (JobFailed $event) {
    // Job failed
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/queues)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
