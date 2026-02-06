# Testing: Getting Started - Laravel 12.x

## Introduction

Laravel is built with testing in mind. Support for both [Pest](https://pestphp.com) and [PHPUnit](https://phpunit.de) is included out of the box, with a `phpunit.xml` file already configured for your application.

Your application's `tests` directory contains two subdirectories:

- **Unit tests**: Focus on small, isolated portions of code (typically single methods). These tests do not boot your Laravel application and cannot access your database or framework services.
- **Feature tests**: Test larger portions of code, including how objects interact with each other or full HTTP requests. Most of your tests should be feature tests as they provide the most confidence that your system is functioning as intended.

Run tests with:
```bash
vendor/bin/pest
vendor/bin/phpunit
php artisan test
```

## Environment

When running tests, Laravel automatically sets the configuration environment to `testing` (via `phpunit.xml`). Session and cache are automatically configured to the `array` driver so no data persists during testing.

### The `.env.testing` Environment File

Create a `.env.testing` file in your project root. It will be used instead of `.env` when running Pest and PHPUnit tests.

## Creating Tests

```bash
# Create a Feature test (default)
php artisan make:test UserTest

# Create a Unit test
php artisan make:test UserTest --unit
```

### Example Tests

**Pest:**
```php
<?php

test('basic', function () {
    expect(true)->toBeTrue();
});
```

**PHPUnit:**
```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```

## Running Tests

### Basic Execution

```bash
# Using Pest
./vendor/bin/pest

# Using PHPUnit
./vendor/bin/phpunit

# Using Laravel's test runner (recommended)
php artisan test

# Pass arguments to underlying test framework
php artisan test --testsuite=Feature --stop-on-failure
```

### Running Tests in Parallel

```bash
# Install ParaTest
composer require brianium/paratest --dev

# Run tests in parallel
php artisan test --parallel

# Specify number of processes
php artisan test --parallel --processes=4
```

#### Parallel Testing and Databases

Laravel automatically creates and migrates test databases for each parallel process. Test databases are suffixed with a unique process token (e.g., `your_db_test_1`, `your_db_test_2`).

Recreate them with:
```bash
php artisan test --parallel --recreate-databases
```

#### Parallel Testing Hooks

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\ParallelTesting;
use Illuminate\Support\ServiceProvider;
use PHPUnit\Framework\TestCase;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        ParallelTesting::setUpProcess(function (int $token) {
            // ...
        });

        ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
            Artisan::call('db:seed');
        });

        ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::tearDownProcess(function (int $token) {
            // ...
        });
    }
}
```

### Reporting Test Coverage

**Requirements**: [Xdebug](https://xdebug.org) or [PCOV](https://pecl.php.net/package/pcov)

```bash
php artisan test --coverage
```

#### Enforcing Minimum Coverage Threshold

```bash
php artisan test --coverage --min=80.3
```

### Profiling Tests

List your slowest tests:
```bash
php artisan test --profile
```

## Configuration Caching

Use the `WithCachedConfig` trait to cache configuration once and reuse it across all tests:

**Pest:**
```php
<?php

use Illuminate\Foundation\Testing\WithCachedConfig;

pest()->use(WithCachedConfig::class);
```

**PHPUnit:**
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithCachedConfig;
use Tests\TestCase;

class ConfigTest extends TestCase
{
    use WithCachedConfig;
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/testing)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
