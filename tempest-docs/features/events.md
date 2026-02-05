# Event Bus

## Overview

An event bus is a synchronous communication system that allows different parts of an application to interact while being decoupled from each other. In Tempest, events can be scalar values or data classes, with handlers being closures or class methods.

## Defining Events

Events are typically simple data classes storing relevant information without logic:

```php
final readonly class AircraftRegistered
{
    public function __construct(
        public string $registration,
    ) {}
}
```

Alternatively, use scalar values like enumerations:

```php
enum AircraftLifecycle
{
    case REGISTERED;
    case RETIRED;
}
```

## Dispatching Events

Use the `EventBus` interface's `dispatch()` method by injecting it as a dependency:

```php
use Tempest\EventBus\EventBus;

final readonly class AircraftService
{
    public function __construct(
        public EventBus $eventBus,
    ) {}

    public function register(Aircraft $aircraft): void
    {
        $this->eventBus->dispatch(new AircraftRegistered(
            registration: $aircraft->icao_code,
        ));
    }
}
```

Alternatively, use the `\Tempest\event()` function for service location.

## Handling Events

### Global Handlers

Use the `#[EventHandler]` attribute for application-wide event listening:

```php
final readonly class AircraftObserver
{
    #[EventHandler]
    public function onAircraftRegistered(AircraftRegistered $event): void
    {
        // …
    }
}
```

### Local Handlers

Register listeners only when needed using the `listen()` method:

```php
final readonly class SyncUsersCommand
{
    public function __construct(
        private readonly EventBus $eventBus,
    ) {}

    #[ConsoleCommand('users:sync')]
    public function __invoke(): void
    {
        $this->eventBus->listen(function (UserSynced $event) {
            // Handle event
        });

        $this->userService->synchronize();
    }
}
```

## Event Middleware

Implement `EventBusMiddleware` to process events before/after handling:

```php
use Tempest\EventBus\EventBusMiddleware;
use Tempest\EventBus\EventBusMiddlewareCallable;

final readonly class EventLoggerMiddleware implements EventBusMiddleware
{
    public function __construct(
        private Logger $logger,
    ) {}

    public function __invoke(string|object $event, EventBusMiddlewareCallable $next): void
    {
        $next($event);

        if ($event instanceof ShouldBeLogged) {
            $this->logger->info($event->getLogMessage());
        }
    }
}
```

### Middleware Priority

Control execution order using the `#[Priority]` attribute with constants: `Priority::FRAMEWORK`, `Priority::HIGHEST`, `Priority::HIGH`, `Priority::NORMAL`, `Priority::LOW`, `Priority::LOWEST`.

### Middleware Discovery

Prevent auto-discovery with `#[SkipDiscovery]` attribute.

## Stopping Event Propagation

Use `#[StopsPropagation]` on events or handlers to limit handling to a single handler:

```php
#[StopsPropagation]
final class MyEvent {}

final class MyHandler
{
    #[EventHandler]
    #[StopsPropagation]
    public function handle(OtherEvent $event): void
    {
        // …
    }
}
```

## Built-in Framework Events

Tempest provides framework events via `KernelEvent` enum (`BOOTED`, `SHUTDOWN`) and migration-related events (`MigrationMigrated`, `MigrationRolledBack`, `MigrationFailed`, `MigrationValidationFailed`).

## Testing

Extend `IntegrationTest` to access event bus testing utilities:

```php
// Record dispatched events
$this->eventBus->recordEventDispatches();

// Prevent event handling
$this->eventBus->preventEventHandling();

// Assertions
$this->eventBus->assertDispatched(AircraftRegistered::class);
$this->eventBus->assertDispatched(AircraftRegistered::class, count: 2);
$this->eventBus->assertDispatched(
    event: AircraftRegistered::class,
    callback: fn (AircraftRegistered $event) => $event->registration === 'LX-JFA'
);
$this->eventBus->assertNotDispatched(AircraftRegistered::class);
$this->eventBus->assertListeningTo(AircraftRegistered::class);
```

### Testing Method-Based Handlers

Resolve handlers from the container and call them directly:

```php
$this->eventBus->preventEventHandling();
$observer = $this->container->get(AircraftObserver::class);
$observer->onAircraftRegistered(new AircraftRegistered(registration: 'LX-JFA'));
```
