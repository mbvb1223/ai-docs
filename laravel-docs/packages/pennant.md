# Laravel 12.x Pennant Feature Flags Documentation

## Introduction

Laravel Pennant is a lightweight feature flag package for incremental rollouts, A/B testing, and trunk-based development.

## Installation

```bash
composer require laravel/pennant
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
php artisan migrate
```

## Defining Features

### Closure-Based

```php
use Laravel\Pennant\Feature;
use Illuminate\Support\Lottery;

Feature::define('new-api', fn (User $user) => match (true) {
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

### Class-Based

```bash
php artisan pennant:feature NewApi
```

```php
namespace App\Features;

class NewApi
{
    public function resolve(User $user): mixed
    {
        return $user->isInternalTeamMember();
    }
}
```

## Checking Features

```php
Feature::active('new-api');
Feature::for($user)->active('new-api');
Feature::active(NewApi::class);

Feature::allAreActive(['new-api', 'site-redesign']);
Feature::someAreActive(['new-api', 'site-redesign']);
Feature::inactive('new-api');
```

### Conditional Execution

```php
Feature::when(NewApi::class,
    fn () => $this->resolveNewApiResponse($request),
    fn () => $this->resolveLegacyApiResponse($request),
);
```

### HasFeatures Trait

```php
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;
}

$user->features()->active('new-api');
```

### Blade Directives

```blade
@feature('site-redesign')
    <!-- Feature is active -->
@else
    <!-- Feature is inactive -->
@endfeature
```

### Middleware

```php
Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api'));
```

## Rich Feature Values

```php
Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
]));

$color = Feature::value('purchase-button');
```

## Updating Features

```php
Feature::activate('new-api');
Feature::deactivate('billing-v2');
Feature::activate('purchase-button', 'seafoam-green');
Feature::forget('purchase-button');

Feature::activateForEveryone('new-api');
Feature::deactivateForEveryone('new-api');
```

## Eager Loading

```php
Feature::for($users)->load(['notifications-beta']);
Feature::for($users)->loadAll();
```

## Testing

```php
Feature::define('purchase-button', 'seafoam-green');
expect(Feature::value('purchase-button'))->toBe('seafoam-green');
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/pennant)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
