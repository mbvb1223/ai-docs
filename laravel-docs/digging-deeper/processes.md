# Laravel 12.x Processes Documentation

## Introduction

Laravel provides an expressive API around the Symfony Process component for invoking external processes.

## Invoking Processes

### Basic Usage

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

### Inspecting Results

```php
$result = Process::run('ls -la');

$result->command();
$result->successful();
$result->failed();
$result->output();
$result->errorOutput();
$result->exitCode();
```

### Throwing Exceptions

```php
$result = Process::run('ls -la')->throw();
$result = Process::run('ls -la')->throwIf($condition);
```

## Process Options

### Working Directory

```php
$result = Process::path(__DIR__)->run('ls -la');
```

### Input

```php
$result = Process::input('Hello World')->run('cat');
```

### Timeouts

```php
$result = Process::timeout(120)->run('bash import.sh');
$result = Process::forever()->run('bash import.sh');
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

### Environment Variables

```php
$result = Process::env(['IMPORT_PATH' => __DIR__])->run('bash import.sh');
```

### TTY Mode

```php
Process::forever()->tty()->run('vim');
```

## Process Output

### Real-time Output

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

### Checking Output

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

### Disabling Output

```php
$result = Process::quietly()->run('bash import.sh');
```

## Process Pipelines

```php
use Illuminate\Process\Pipe;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

// Simple array syntax
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

## Asynchronous Processes

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // Do other work
}

$result = $process->wait();
```

### Process IDs and Signals

```php
$pid = $process->id();
$process->signal(SIGUSR2);
```

### Real-time Output

```php
$process = Process::start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    sleep(1);
}
```

### Wait Until Condition

```php
$process->waitUntil(function (string $type, string $output) {
    return $output === 'Ready...';
});
```

## Concurrent Processes

```php
use Illuminate\Process\Pool;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start();

$results = $pool->wait();
```

### Convenient Concurrent Execution

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});
```

### Named Pool Processes

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
});

$results = $pool->wait();
return $results['first']->output();
```

## Testing

### Faking Processes

```php
Process::fake();

$response = $this->get('/import');

Process::assertRan('bash import.sh');
```

### Faking Specific Processes

```php
Process::fake([
    'cat *' => Process::result(output: 'Test output'),
    'ls *' => 'Test "ls" output',
]);
```

### Assertions

```php
Process::assertRan('ls -la');
Process::assertDidntRun('ls -la');
Process::assertRanTimes('ls -la', times: 3);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/processes)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
