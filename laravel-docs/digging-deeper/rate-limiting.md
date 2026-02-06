# Laravel 12.x Rate Limiting Documentation

## Introduction

Laravel includes a simple-to-use rate limiting abstraction using your application's cache.

## Cache Configuration

```php
'default' => env('CACHE_STORE', 'database'),
'limiter' => 'redis',
```

## Basic Usage

### Simple Attempt

```php
use Illuminate\Support\Facades\RateLimiter;

$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perMinute = 5,
    function() {
        // Send message...
    }
);

if (! $executed) {
    return 'Too many messages sent!';
}
```

### Custom Decay Rate

```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // Send message...
    },
    $decayRate = 120,
);
```

## Manually Incrementing Attempts

### Checking for Too Many Attempts

```php
if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return 'Too many attempts!';
}

RateLimiter::increment('send-message:'.$user->id);
```

### Checking Remaining Attempts

```php
if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);
    // Send message...
}
```

### Incrementing by Custom Amount

```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

## Determining Limiter Availability

```php
if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);
    return 'You may try again in '.$seconds.' seconds.';
}
```

## Clearing Attempts

```php
RateLimiter::clear('send-message:'.$message->user_id);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/rate-limiting)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
