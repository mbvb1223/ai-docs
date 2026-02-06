# Laravel 12.x Mocking Documentation

## Introduction

Laravel provides helpful methods for mocking events, jobs, and facades. These primarily provide a convenience layer over Mockery.

## Mocking Objects

### Basic Mock Example

```php
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
});
```

### Using the `mock` Helper

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

### Partial Mocking

```php
$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

### Spying on Objects

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

## Mocking Facades

```php
use Illuminate\Support\Facades\Cache;

test('get index', function () {
    Cache::expects('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/users');
});
```

### Facade Spies

```php
use Illuminate\Support\Facades\Cache;

test('values are stored in cache', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
});
```

## Interacting With Time

### Time Travel Methods

```php
test('time can be manipulated', function () {
    // Travel into the future
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Travel into the past
    $this->travel(-5)->hours();

    // Travel to an explicit time
    $this->travelTo(now()->minus(hours: 6));

    // Return to the present
    $this->travelBack();
});
```

### Time Travel with Closures

```php
$this->travel(5)->days(function () {
    // Test something five days into the future
});

$this->travelTo(now()->minus(days: 10), function () {
    // Test something during a given moment
});
```

### Freezing Time

```php
use Illuminate\Support\Carbon;

$this->freezeTime(function (Carbon $time) {
    // ...
});

$this->freezeSecond(function (Carbon $time) {
    // ...
});
```

### Real-World Example

```php
use App\Models\Thread;

test('forum threads lock after one week of inactivity', function () {
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    expect($thread->isLockedByInactivity())->toBeTrue();
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/mocking)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
