# Processes

## Overview

Tempest provides a testable wrapper around the Symfony Process component, inspired by Laravel's own wrapper. It allows you to run one or multiple processes synchronously or asynchronously, while being testable and convenient to use.

## Synchronous Processes

The `Tempest\Process\ProcessExecutor` interface serves as the entry point for invoking processes. It offers a `run()` method for synchronous execution and a `start()` method for asynchronous execution. You can access this interface by injecting it as a dependency in your classes.

### Example Usage

```php
use Tempest\Process\ProcessExecutor;

final readonly class Composer
{
    public function __construct(
        private ProcessExecutor $executor
    ) {}

    public function update(): void
    {
        $this->executor->run('composer update');
    }
}
```

The `run()` method returns a `ProcessResult` instance containing:
- `successful()` — boolean indicating success
- `failed()` — boolean indicating failure
- `exitCode` — the process exit code
- `output` — standard output
- `errorOutput` — error output

## Asynchronous Processes

Use the `start()` method to run processes asynchronously, which returns an `InvokedProcess` instance for monitoring. Available methods include:

- `signal()` — send a signal to the running process
- `stop()` — terminate the process
- `wait()` — wait for completion with output callback

```php
$this->executor
    ->start('composer update')
    ->wait(function (OutputChannel $channel, string $output) {
        echo $output;
    });
```

## Process Pools

Execute multiple tasks simultaneously using process pools. Call `pool()` on `ProcessExecutor` to get a `Pool` instance:

```php
$pool = $this->executor->pool([
    'composer update',
    'bun install',
]);

$pool->start();
$pool->count();
$pool->forEach(fn (InvokedProcess $process) => /** ... */);
$pool->forEachRunning(fn (InvokedProcess $process) => /** ... */);
$pool->signal(SIGINT);
$pool->stop();
```

For output-focused operations, use `concurrently()` with destructuring:

```php
[$composer, $bun] = $this->executor->concurrently([
    'composer update',
    'bun install',
]);
```

## Testing

### Mocking Process Results

Access testing utilities via the `process` property in `IntegrationTest`:

```php
$this->process->mockProcessResult('composer up');

// Call application code...

$this->process->assertCommandRan('composer up');
$this->process->assertRan(function (PendingProcess $process, ProcessResult $result) {
    // assertions...
});
```

### Describing Asynchronous Processes

Use `describe()` to define process expectations:

```php
$this->process->mockProcessResults([
    'composer up' => $this->process
        ->describe()
        ->iterations(1)
        ->output('Nothing to install, update or remove'),
    'bun install' => $this->process
        ->describe()
        ->iterations(4)
        ->output('Checked 225 installs across 274 packages (no changes) [144.00ms]'),
]);
```

### Allowing Actual Process Execution

By default, non-mocked processes throw exceptions to prevent side effects. Enable actual execution with:

```php
$this->process->allowRunningActualProcesses();
```
