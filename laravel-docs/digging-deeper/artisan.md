# Laravel 12.x Artisan Console Documentation

## Introduction

Artisan is Laravel's command-line interface for application development.

```bash
php artisan list
php artisan help migrate
```

### Tinker (REPL)

```bash
composer require laravel/tinker
php artisan tinker
```

## Writing Commands

### Generating Commands

```bash
php artisan make:command SendEmails
```

### Command Structure

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    protected $signature = 'mail:send {user}';
    protected $description = 'Send a marketing email to a user';

    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

### Closure Commands

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("Sending email to: {$user}!");
})->purpose('Send a marketing email to a user');
```

### Isolatable Commands

```php
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // Only one instance runs at a time
}
```

## Defining Input Expectations

### Arguments

```php
protected $signature = 'mail:send {user}';      // Required
protected $signature = 'mail:send {user?}';     // Optional
protected $signature = 'mail:send {user=foo}';  // With default
```

### Options

```php
protected $signature = 'mail:send {user} {--queue}';       // Boolean
protected $signature = 'mail:send {user} {--queue=}';      // With value
protected $signature = 'mail:send {user} {--queue=default}'; // With default
protected $signature = 'mail:send {user} {--Q|queue}';     // Shortcut
```

### Input Arrays

```php
'mail:send {user*}'           // Multiple arguments
'mail:send {--id=*}'          // Multiple option values
```

### Input Descriptions

```php
protected $signature = 'mail:send
                        {user : The ID of the user}
                        {--queue : Whether the job should be queued}';
```

## Command I/O

### Retrieving Input

```php
$userId = $this->argument('user');
$arguments = $this->arguments();
$queueName = $this->option('queue');
$options = $this->options();
```

### Prompting for Input

```php
$name = $this->ask('What is your name?');
$name = $this->ask('What is your name?', 'Taylor');
$password = $this->secret('What is the password?');

if ($this->confirm('Do you wish to continue?')) {
    // ...
}

$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);
```

### Writing Output

```php
$this->info('The command was successful!');    // Green
$this->error('Something went wrong!');          // Red
$this->line('Display this on the screen');     // Plain
$this->newLine();                               // Blank line
$this->newLine(3);                              // Three blank lines
```

### Tables

```php
$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

### Progress Bars

```php
$users = $this->withProgressBar(User::all(), function (User $user) {
    $this->performTask($user);
});
```

## Registering Commands

Commands in `app/Console/Commands` are registered automatically. Add additional directories:

```php
->withCommands([
    __DIR__.'/../app/Domain/Orders/Commands',
])
```

## Programmatically Executing Commands

```php
use Illuminate\Support\Facades\Artisan;

$exitCode = Artisan::call('mail:send', [
    'user' => 1, '--queue' => 'default'
]);

Artisan::call('mail:send 1 --queue=default');

// Queue command
Artisan::queue('mail:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

### From Other Commands

```php
public function handle(): void
{
    $this->call('mail:send', ['user' => 1, '--queue' => 'default']);
    $this->callSilently('mail:send', ['user' => 1]);
}
```

## Signal Handling

```php
public function handle(): void
{
    $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

    while ($this->shouldKeepRunning) {
        // ...
    }
}
```

## Stub Customization

```bash
php artisan stub:publish
```

## Events

- `ArtisanStarting` - When Artisan starts
- `CommandStarting` - Before a command runs
- `CommandFinished` - After a command finishes

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/artisan)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
