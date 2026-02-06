# Laravel 12.x Prompts Documentation

## Introduction

Laravel Prompts adds beautiful, user-friendly forms to command-line applications.

## Installation

```bash
composer require laravel/prompts
```

## Available Prompts

### Text

```php
use function Laravel\Prompts\text;

$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.',
    required: true,
    validate: fn (string $value) => strlen($value) < 3
        ? 'The name must be at least 3 characters.'
        : null
);
```

### Textarea

```php
use function Laravel\Prompts\textarea;

$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

### Password

```php
use function Laravel\Prompts\password;

$password = password(
    label: 'What is your password?',
    required: true
);
```

### Confirm

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false
);
```

### Select

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner'
);
```

### Multi-select

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    required: true
);
```

### Search

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user',
    options: fn (string $value) => User::whereLike('name', "%{$value}%")
        ->pluck('name', 'id')->all()
);
```

### Suggest

```php
use function Laravel\Prompts\suggest;

$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle']
);
```

## Forms

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->password('What is your password?', name: 'password')
    ->confirm('Do you accept the terms?')
    ->submit();
```

## Informational Messages

```php
use function Laravel\Prompts\{info, note, warning, error, alert};

info('Package installed successfully.');
warning('This is a warning.');
error('An error occurred.');
```

## Tables

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

## Spinner

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Fetching response...'
);
```

## Progress Bars

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/prompts)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
