# Laravel 12.x Console Testing Documentation

## Introduction

Laravel provides a simple API for testing custom console commands.

## Success / Failure Expectations

### Testing Exit Codes

```php
test('console command', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```

### Exit Code Assertions

- `assertExitCode(code)` - Assert the command exited with a specific code
- `assertNotExitCode(code)` - Assert the command did NOT exit with a specific code
- `assertSuccessful()` - Assert the command exited successfully (code 0)
- `assertFailed()` - Assert the command failed (non-zero exit code)

```php
$this->artisan('inspire')->assertSuccessful();
$this->artisan('inspire')->assertFailed();
```

## Input / Output Expectations

### Testing Questions and Responses

```php
test('console command', function () {
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
});
```

### Testing Search Prompts

```php
$this->artisan('example')
    ->expectsSearch('What is your name?', search: 'Tay', answers: [
        'Taylor Otwell',
        'Taylor Swift',
        'Darian Taylor'
    ], answer: 'Taylor Otwell')
    ->assertExitCode(0);
```

### Output Assertions

- `expectsOutput(string)` - Assert the command outputs specific text
- `doesntExpectOutput(string)` - Assert the command does NOT output text
- `doesntExpectOutput()` - Assert the command produces no output
- `expectsOutputToContain(string)` - Assert output contains a substring
- `doesntExpectOutputToContain(string)` - Assert output doesn't contain a substring

### Confirmation Expectations

```php
$this->artisan('module:import')
    ->expectsConfirmation('Do you really wish to run this command?', 'no')
    ->assertExitCode(1);
```

### Table Expectations

```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'Email',
    ], [
        [1, '[email protected]'],
        [2, '[email protected]'],
    ]);
```

## Console Events

Enable `CommandStarting` and `CommandFinished` events in tests:

```php
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/console-tests)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
