# Laravel 12.x Events Documentation

## Introduction

Laravel's events provide a simple observer pattern implementation for subscribing to and listening for various events within your application. Event classes are stored in `app/Events` and listeners in `app/Listeners`.

## Generating Events and Listeners

```bash
php artisan make:event PodcastProcessed
php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

Or without arguments for interactive prompts:
```bash
php artisan make:event
php artisan make:listener
```

## Registering Events and Listeners

### Event Discovery (Default)

Laravel automatically scans the `Listeners` directory and registers methods beginning with `handle` or `__invoke` that type-hint an event:

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    public function handle(PodcastProcessed $event): void
    {
        // Handle the event
    }
}
```

**Listen to multiple events using union types:**
```php
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

**Configure custom listener directories in `bootstrap/app.php`:**
```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])

// With wildcards
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```

**List all registered listeners:**
```bash
php artisan event:list
```

### Manually Registering Events

In `AppServiceProvider`:

```php
use App\Events\PodcastProcessed;
use App\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

### Closure Listeners

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // Handle the event
    });
}
```

**Queueable Anonymous Listeners:**
```php
use function Illuminate\Events\queueable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->plus(seconds: 10)));

// With error handling
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // Handle failure
}));
```

**Wildcard Listeners:**
```php
Event::listen('event.*', function (string $eventName, array $data) {
    // Handle any event matching the pattern
});
```

## Defining Events

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}
}
```

## Defining Listeners

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    public function __construct() {}

    public function handle(OrderShipped $event): void
    {
        // Access the order using $event->order
    }
}
```

**Stop event propagation** by returning `false`:
```php
public function handle(OrderShipped $event): void
{
    if ($condition) {
        return false;
    }
}
```

## Queued Event Listeners

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // Listener will be queued automatically
}
```

### Customizing Queue Settings

```php
class SendShipmentNotification implements ShouldQueue
{
    public $connection = 'sqs';
    public $queue = 'listeners';
    public $delay = 60;
}
```

**Or using methods:**
```php
public function viaConnection(): string
{
    return 'sqs';
}

public function viaQueue(): string
{
    return 'listeners';
}

public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

### Conditionally Queueing

```php
class RewardGiftCard implements ShouldQueue
{
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

### Manually Interacting With the Queue

```php
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

### Database Transactions

Implement `ShouldQueueAfterCommit` to dispatch after database transactions commit:

```php
use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

### Queued Listener Middleware

```php
class SendShipmentNotification implements ShouldQueue
{
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }

    public function middleware(OrderShipped $event): array
    {
        return [new RateLimited];
    }
}
```

### Encrypted Queued Listeners

```php
use Illuminate\Contracts\Queue\ShouldBeEncrypted;

class SendShipmentNotification implements ShouldQueue, ShouldBeEncrypted
{
    // Data automatically encrypted before queuing
}
```

### Unique Event Listeners

```php
use Illuminate\Contracts\Queue\ShouldBeUnique;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    public $uniqueFor = 3600;

    public function __invoke(LicenseSaved $event): void
    {
        // Only one instance queued at a time
    }

    public function uniqueId(LicenseSaved $event): string
    {
        return 'listener:'.$event->license->id;
    }

    public function uniqueVia(LicenseSaved $event): Repository
    {
        return Cache::driver('redis');
    }
}
```

### Handling Failed Jobs

```php
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(OrderShipped $event): void
    {
        // ...
    }

    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // Handle failure
    }
}
```

**Configure retry attempts:**
```php
class SendShipmentNotification implements ShouldQueue
{
    public $tries = 5;

    // Or with timeout
    public function retryUntil(): DateTime
    {
        return now()->plus(minutes: 5);
    }
}
```

**Configure backoff:**
```php
public $backoff = 3;

// Or with method
public function backoff(OrderShipped $event): int
{
    return 3;
}

// Exponential backoff
public function backoff(OrderShipped $event): array
{
    return [1, 5, 10];
}
```

## Dispatching Events

```php
OrderShipped::dispatch($order);

// Conditional dispatch
OrderShipped::dispatchIf($condition, $order);
OrderShipped::dispatchUnless($condition, $order);
```

### After Database Transactions

Implement `ShouldDispatchAfterCommit`:

```php
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}
}
```

### Deferring Events

```php
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);
    $user->posts()->create(['title' => 'My first post!']);
});

// Defer specific events
Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);
    $user->posts()->create(['title' => 'My first post!']);
}, ['eloquent.created: '.User::class]);
```

## Event Subscribers

### Writing Event Subscribers

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    public function handleUserLogin(Login $event): void {}

    public function handleUserLogout(Logout $event): void {}

    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            Login::class,
            [UserEventSubscriber::class, 'handleUserLogin']
        );

        $events->listen(
            Logout::class,
            [UserEventSubscriber::class, 'handleUserLogout']
        );
    }
}
```

**Or with array return:**
```php
public function subscribe(Dispatcher $events): array
{
    return [
        Login::class => 'handleUserLogin',
        Logout::class => 'handleUserLogout',
    ];
}
```

### Registering Event Subscribers

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

## Testing

### Basic Event Testing

```php
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // Perform order shipping...

    Event::assertDispatched(OrderShipped::class);
    Event::assertDispatched(OrderShipped::class, 2);
    Event::assertDispatchedOnce(OrderShipped::class);
    Event::assertNotDispatched(OrderFailedToShip::class);
    Event::assertNothingDispatched();
});
```

**With closure assertions:**
```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

**Assert listeners:**
```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

### Faking a Subset of Events

```php
test('orders can be processed', function () {
    Event::fake([OrderCreated::class]);

    $order = Order::factory()->create();
    Event::assertDispatched(OrderCreated::class);

    // Other events dispatch normally
});

// Exclude specific events
Event::fake()->except([OrderCreated::class]);
```

### Scoped Event Fakes

```php
$order = Event::fakeFor(function () {
    $order = Order::factory()->create();
    Event::assertDispatched(OrderCreated::class);
    return $order;
});

// Events dispatch normally outside the closure
$order->update([...]);
```

## Production Optimization

Cache event listeners for faster registration:
```bash
php artisan event:cache
php artisan event:clear
```

This should be part of your deployment process.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/events)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
