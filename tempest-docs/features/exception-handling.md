# Exception Handling

## Overview

Tempest includes an exception handler that enables reporting exceptions and rendering error responses. Custom exception reporters and renderers can be created by implementing the `ExceptionReporter` and `ExceptionRenderer` interfaces respectively. These classes are automatically discovered and require no manual registration.

## Processing Exceptions

The `ExceptionProcessor` interface provides a `process()` method to report exceptions without throwing them, allowing the application to continue execution:

```php
use Tempest\Core\Exceptions\ExceptionProcessor;

final readonly class CreateUser
{
    public function __construct(
        private ExceptionProcessor $exceptions
    ) {}

    public function __invoke(): void
    {
        try {
            // Some code that may throw an exception
        } catch (SomethingFailed $somethingFailed) {
            $this->exceptions->process($somethingFailed);
        }
    }
}
```

## Disabling Exception Logging

The default `LoggingExceptionReporter` is automatically included. To disable it, create an `ExceptionsConfig` configuration file:

```php
use Tempest\Core\Exceptions\ExceptionsConfig;

return new ExceptionsConfig(
    logging: false,
);
```

## Adding Context to Exceptions

Exceptions can provide additional information by implementing the `ProvidesContext` interface:

```php
use Tempest\Core\ProvidesContext;

final readonly class UserWasNotFound extends Exception implements ProvidesContext
{
    public function __construct(private string $userId)
    {
        parent::__construct("User {$userId} not found.");
    }

    public function context(): array
    {
        return [
            'user_id' => $this->userId,
        ];
    }
}
```

## Writing Exception Reporters

Create custom reporters by implementing the `ExceptionReporter` interface with a `report()` method:

```php
use Tempest\Core\Exceptions\ExceptionReporter;
use Throwable;

final class SentryExceptionReporter implements ExceptionReporter
{
    public function __construct(
        private SentryClient $sentry,
    ) {}

    public function report(Throwable $throwable): void
    {
        $this->sentry->captureException($throwable);
    }
}
```

Reporters are automatically discovered and all registered reporters are invoked for each exception. If a reporter throws an exception, it is silently caught to prevent infinite loops.

### Accessing Exception Context

Reporters can leverage context from exceptions implementing `ProvidesContext`:

```php
use Tempest\Core\Exceptions\ExceptionReporter;
use Tempest\Core\ProvidesContext;
use Sentry\State\HubInterface as Sentry;
use Sentry\State\Scope;

final class SentryExceptionReporter implements ExceptionReporter
{
    public function __construct(
        private readonly Sentry $sentry,
    ) {}

    public function report(Throwable $throwable): void
    {
        $this->sentry->withScope(function (Scope $scope) use ($throwable) {
            if ($throwable instanceof ProvidesContext) {
                $scope->withContext($throwable->context());
            }

            $scope->captureException($throwable);
        });
    }
}
```

### Conditional Reporting

Reporters can implement conditional logic to report only specific exception types:

```php
use Tempest\Core\Exceptions\ExceptionReporter;
use Throwable;

final class CriticalErrorReporter implements ExceptionReporter
{
    public function __construct(
        private AlertService $alerts,
    ) {}

    public function report(Throwable $throwable): void
    {
        if (! $throwable instanceof CriticalException) {
            return;
        }

        $this->alerts->sendCriticalAlert(
            message: $throwable->getMessage(),
        );
    }
}
```

## Customizing Exception Rendering

Create custom renderers by implementing `ExceptionRenderer` with `canRender()` and `render()` methods:

```php
use Tempest\Http\ContentType;
use Tempest\Http\HttpRequestFailed;
use Tempest\Http\Request;
use Tempest\Http\Response;
use Tempest\Http\Responses\NotFound;
use Tempest\Http\Status;
use Tempest\Router\Exceptions\ExceptionRenderer;
use Throwable;

use function Tempest\View\view;

final class NotFoundExceptionRenderer implements ExceptionRenderer
{
    public function canRender(Throwable $throwable, Request $request): bool
    {
        if (! $request->accepts(ContentType::HTML)) {
            return false;
        }

        if (! $throwable instanceof HttpRequestFailed) {
            return false;
        }

        return $throwable->status === Status::NOT_FOUND;
    }

    public function render(Throwable $throwable): Response
    {
        return new NotFound(
            body: view('./404.view.php'),
        );
    }
}
```

Exception renderers are automatically discovered and checked in `#[Priority]` order.

## Testing

Extend `Tempest\Framework\Testing\IntegrationTest` to access exception testing utilities:

```php
// Allows exceptions to be processed during tests
$this->exceptions->allowProcessing();

// Assert that the exception was processed
$this->exceptions->assertProcessed(UserNotFound::class);

// Assert that the exception was not processed
$this->exceptions->assertNotProcessed(UserNotFound::class);

// Assert that no exceptions were processed
$this->exceptions->assertNothingProcessed();
```

By default, exception processing is disabled during tests. It is recommended to unit-test custom `ExceptionReporter` implementations.
