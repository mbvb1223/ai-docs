# Testing

## Overview

Tempest is built with testing in mind. It ships with convenient utilities that make it easy to test application code without boilerplate. The framework integrates PHPUnit through the `IntegrationTest` test case class, which boots the framework with testing-appropriate configuration and provides access to testing utilities.

## Running Tests

Test classes requiring framework interaction should extend `IntegrationTest`. Tempest includes a default `phpunit.xml` configuration that discovers tests in the `tests` directory.

Execute tests with:
```bash
./vendor/bin/phpunit
```

## Using the Database

Tests don't interact with the database by default. Enable database functionality using the `setup()` method on the database testing utility:

```php
final class ShowAircraftControllerTest extends IntegrationTest
{
    #[PreCondition]
    protected function configure(): void
    {
        $this->database->setup();
    }
}
```

The `PreCondition` attribute runs the method after PHPUnit's `setUp()` method.

### Running Migrations

Run specific migrations instead of all migrations:

```php
$this->database->migrate(
    CreateMigrationsTable::class,
    CreateAircraftTable::class,
);
```

### Dedicated Testing Database

Create a `database.testing.config.php` file to isolate tests:

```php
use Tempest\Database\Config\SQLiteConfig;

return new SQLiteConfig(
    path: __DIR__ . '/testing.sqlite'
);
```

## Spoofing the Environment

Override the `Environment` singleton to test environment-specific behavior:

```php
use Tempest\Core\Environment;

$this->container->singleton(Environment::class, Environment::PRODUCTION);
```

## Changing Test Location

Modify the `phpunit.xml` testsuite configuration to change where PHPUnit discovers tests:

```xml
<testsuite name="Tests">
    <directory suffix="Test.php">./app</directory>
</testsuite>
```

## Discovering Test-Specific Fixtures

Override `discoverTestLocations()` to customize fixture discovery:

```php
protected function discoverTestLocations(): array
{
    return [
        new DiscoveryLocation('Tests\\Aircraft', __DIR__ . '/Aircraft'),
    ];
}
```

## Using Pest as a Test Runner

Pest provides a functional testing approach built on PHPUnit. Replace PHPUnit with Pest:

```bash
composer remove phpunit/phpunit
composer require pestphp/pest --dev --with-all-dependencies
```

Create `tests/Pest.php`:

```php
pest()
    ->extend(Tests\IntegrationTest::class)
    ->in(__DIR__);
```

Run tests with:
```bash
./vendor/bin/pest
```
