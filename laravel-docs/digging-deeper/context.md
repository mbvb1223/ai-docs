# Laravel 12.x Context Documentation

## Introduction

Laravel's "context" capabilities enable you to capture, retrieve, and share information throughout requests, jobs, and commands. Captured information is automatically included in logs.

## Capturing Context

### Basic Methods

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');

Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);

// Only add if key doesn't exist
Context::addIf('key', 'second');

// Increment/decrement
Context::increment('records_added');
Context::decrement('records_added', 5);
```

### Conditional Context

```php
Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

### Scoped Context

```php
Context::add('trace_id', 'abc-999');

Context::scope(
    function () {
        Context::add('action', 'adding_friend');
        Log::debug("Action performed");
    },
    data: ['user_name' => 'taylor_otwell'],
    hidden: ['user_id' => 987],
);
```

### Stacks

```php
Context::push('breadcrumbs', 'first_value');
Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// ['first_value', 'second_value', 'third_value']

// Check if value exists in stack
Context::stackContains('breadcrumbs', 'first_value'); // true
```

## Retrieving Context

```php
$value = Context::get('key');

$data = Context::only(['first_key', 'second_key']);
$data = Context::except(['first_key']);

// Retrieve and remove
$value = Context::pull('key');

// Pop from stack
Context::pop('breadcrumbs');

// Remember with fallback
$permissions = Context::remember('user-permissions', fn () => $user->permissions);

// Get all data
$data = Context::all();
```

### Checking Existence

```php
if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

## Removing Context

```php
Context::forget('first_key');
Context::forget(['first_key', 'second_key']);
```

## Hidden Context

Store data not appended to logs:

```php
Context::addHidden('key', 'value');
Context::getHidden('key');  // 'value'
Context::get('key');        // null
```

Hidden counterparts available for all methods:
- `addHidden()`, `addHiddenIf()`, `pushHidden()`
- `getHidden()`, `pullHidden()`, `popHidden()`
- `onlyHidden()`, `exceptHidden()`, `allHidden()`
- `hasHidden()`, `missingHidden()`, `forgetHidden()`

## Events

### Dehydrating (when job is dispatched)

```php
Context::dehydrating(function (Repository $context) {
    $context->addHidden('locale', Config::get('app.locale'));
});
```

### Hydrated (when job begins executing)

```php
Context::hydrated(function (Repository $context) {
    if ($context->hasHidden('locale')) {
        Config::set('app.locale', $context->getHidden('locale'));
    }
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/context)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
