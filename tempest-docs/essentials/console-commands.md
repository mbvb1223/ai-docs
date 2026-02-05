# Console Commands

## Overview

Tempest uses the `#[ConsoleCommand]` attribute for creating console commands. The framework automatically discovers these methods through its discovery system, making them available via the `./tempest` executable.

## Creating Console Commands

Define commands by adding the `#[ConsoleCommand]` attribute to class methods:

```php
final readonly class TrackOperatingAircraft
{
    #[ConsoleCommand(name: 'aircraft:track')]
    public function __invoke(): void
    {
        // …
    }
}
```

Commands are named after the class and method by default, but the `name` parameter allows custom naming.

## Writing to Output

Inject the `Tempest\Console\Console` interface or use the `HasConsole` trait:

```php
$this->console->writeln('Hello from Tempest!');
$this->console->info('Informational message.');
$this->console->error('Error message.');
$this->console->warning('Warning message.');
$this->console->ask('Prompt text', validation: [new Email()]);
$this->console->task('Syncing...', $this->synchronize(...));
```

## Exit Codes

Return integer codes (0-255) or use the `ExitCode` enumeration:

```php
use Tempest\Console\ExitCode;

public function __invoke(): ExitCode
{
    return ExitCode::SUCCESS;
}
```

## Command Arguments

Method parameters define command arguments automatically:

```php
#[ConsoleCommand('aircraft:track')]
public function __invoke(AircraftType $type, ?int $radius = null): void
{
    // Nullable parameters become optional flags
}
```

### Negating Boolean Arguments

Prefix boolean flags with `--no` to negate them. A `$validate` parameter with `true` default becomes `--no-validate` to set it `false`.

### Argument Documentation

Use `#[ConsoleArgument]` to add descriptions and aliases:

```php
#[ConsoleCommand(name: 'aircraft:track', description: 'Updates aircraft data')]
public function __invoke(
    #[ConsoleArgument(description: 'Aircraft type')]
    AircraftType $type,
    #[ConsoleArgument(description: 'Search radius', aliases: ['r'])]
    ?int $radius = null
): void
{
    // …
}
```

## Configuring Commands

### Description

```php
#[ConsoleCommand(description: 'Updates operating aircraft in the database')]
public function __invoke(): void { }
```

### Hiding Commands

```php
#[ConsoleCommand(hidden: true)]
public function __invoke(): void { }
```

Hidden commands remain invokable but don't appear in command listings.

### Aliases

```php
#[ConsoleCommand('aircraft:track', aliases: ['track'])]
public function __invoke(): void { }
```

### Production Safety

Prevent execution in production using `CautionMiddleware`:

```php
#[ConsoleCommand('aircraft:sync', middleware: [CautionMiddleware::class])]
public function __invoke(): void { }
```

## Interactive Components

Available console interactions include:
- `$console->ask()` — Prompts with validation
- `$console->confirm()` — Yes/no prompts
- `$console->password()` — Masked input
- `$console->progressBar()` — Progress visualization
- `$console->search()` — Real-time search with results
- `$console->task()` — Task execution with progress

Interactive components only work on Mac and Linux; Windows uses non-interactive fallbacks.

## Shell Completion

Install completions with:

```bash
./tempest completion:install
```

For Zsh, update `~/.zshrc`:
```bash
fpath=(~/.zsh/completions $fpath)
autoload -Uz compinit && compinit
```

For Bash, add to `~/.bashrc`:
```bash
source ~/.bash_completion.d/tempest.bash
```

Additional commands: `completion:show` and `completion:uninstall`.

## Middleware

Implement `Tempest\Console\ConsoleMiddleware` for custom middleware:

```php
final readonly class InspireMiddleware implements ConsoleMiddleware
{
    public function __invoke(Invocation $invocation, ConsoleMiddlewareCallable $next): ExitCode|int
    {
        if ($invocation->argumentBag->get('inspire')) {
            $this->console->writeln($this->inspiration->random());
        }
        return $next($invocation);
    }
}
```

### Priority

Control execution order with the `#[Priority]` attribute using `Priority::FRAMEWORK`, `Priority::HIGHEST`, `Priority::HIGH`, `Priority::NORMAL`, `Priority::LOW`, or `Priority::LOWEST`.

### Discovery Control

Use `#[SkipDiscovery]` to prevent automatic global registration.

### Built-in Middleware

- `ForceMiddleware` — Adds `--force` flag
- `CautionMiddleware` — Production safeguard
- `OverviewMiddleware` — Lists commands
- `ResolveOrRescueMiddleware` — Suggests similar commands
- `HelpMiddleware` — Provides `--help` support
- `ConsoleExceptionMiddleware` — Exception rendering

## Scheduling

Use `#[Schedule]` with `Interval` or `Every` values for automated execution:

```php
#[Schedule(Every::HOUR)]
public function syncData(): void { }
```

See the [scheduling documentation](../features/scheduling.md) for details.

## Testing

Test commands using the `console` property in `IntegrationTest`:

```php
$this->console
    ->call(ExportUsersCommand::class)
    ->assertSuccess()
    ->assertSee('12 users exported');
```
