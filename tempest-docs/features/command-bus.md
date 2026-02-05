# Command Bus

## Overview

Tempest includes a built-in command bus for dispatching commands to their handlers. This architecture supports both synchronous and asynchronous execution, middleware support, and offers multiple advantages over a more direct approach: commands and their handlers can easily be tested in isolation, they are simple to serialize.

## Commands and Handlers

### Creating Commands

Commands are straightforward data classes requiring no special interface implementation:

```php
final readonly class CreateUser
{
    public function __construct(
        public string $name,
        public string $email,
        public string $passwordHash,
    ) {}
}
```

### Creating Handlers

Handlers use automatic discovery via the `#[CommandHandler]` attribute. The command type is inferred from the first parameter:

```php
use Tempest\CommandBus\CommandHandler;

final class UserHandlers
{
    #[CommandHandler]
    public function handleCreateUser(CreateUser $createUser): void
    {
        User::create(
            name: $createUser->name,
            email: $createUser->email,
            password: $createUser->passwordHash,
        );
    }
}
```

Handler method names are flexible (`handle()`, `handleCreateUser()`, invokable methods, etc.).

### Dispatching Commands

**Using the command function:**

```php
use function Tempest\command;

command(new CreateUser($name));
```

**Injecting CommandBus:**

```php
use Tempest\CommandBus\CommandBus;

final readonly class UserController
{
    public function __construct(
        private CommandBus $commandBus,
    ) {}

    public function create(): Response
    {
        $this->commandBus->dispatch(new CreateUser($name));
    }
}
```

## Async Commands

Async commands execute in the background outside the main PHP process. Add the `#[Async]` attribute:

```php
use Tempest\CommandBus\Async;

#[Async]
final readonly class SendMail
{
    public function __construct(
        public string $to,
        public string $body,
    ) {}
}
```

Dispatch async commands identically to synchronous ones. To execute them, run `tempest command:monitor` as a long-running daemon process on your server.

**Note:** Async command handling is experimental and not covered by backwards compatibility guarantees.

## Command Bus Middleware

Middleware implements `CommandBusMiddleware` and processes commands before handler execution:

```php
use Tempest\CommandBus\CommandBusMiddleware;
use Tempest\CommandBus\CommandBusMiddlewareCallable;

class MyCommandBusMiddleware implements CommandBusMiddleware
{
    public function __construct(
        private Logger $logger,
    ) {}

    public function __invoke(object $command, CommandBusMiddlewareCallable $next): void
    {
        $next($command);

        if ($command instanceof ShouldBeLogged) {
            $this->logger->info($command->getLogMessage());
        }
    }
}
```

### Middleware Priority

Control execution order with the `#[Priority]` attribute:

```php
use Tempest\Core\Priority;

#[Priority(Priority::HIGH)]
final readonly class MyCommandBusMiddleware implements CommandBusMiddleware
{ /* … */ }
```

Available priority constants: `FRAMEWORK`, `HIGHEST`, `HIGH`, `NORMAL`, `LOW`, `LOWEST`.

### Middleware Discovery

Middleware is auto-discovered by default. Prevent this with `#[SkipDiscovery]`:

```php
use Tempest\Discovery\SkipDiscovery;

#[SkipDiscovery]
final readonly class MyCommandBusMiddleware implements CommandBusMiddleware
{ /* … */ }
```
