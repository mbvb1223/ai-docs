# Laravel 12.x Redis Documentation

## Introduction

Redis is an open source, advanced key-value store often referred to as a data structure server since keys can contain strings, hashes, lists, sets, and sorted sets.

### Installation

Install PhpRedis PHP extension via PECL (recommended) or use Predis:

```bash
composer require predis/predis
```

## Configuration

Configure Redis settings in `config/database.php`:

```php
'redis' => [
    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],
],
```

### Connection URL Format

```php
'default' => [
    'url' => 'tcp://127.0.0.1:6379?database=0',
],

'cache' => [
    'url' => 'tls://user:[email protected]:6380?database=1',
],
```

### TLS/SSL Encryption

```php
'default' => [
    'scheme' => 'tls',
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

### Clusters

```php
'clusters' => [
    'default' => [
        [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
    ],
],
```

### PhpRedis Configuration

```php
'default' => [
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

### Retry and Backoff

```php
'default' => [
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

Supported algorithms: `default`, `decorrelated_jitter`, `equal_jitter`, `exponential`, `uniform`, `constant`

### Serialization and Compression

```php
'options' => [
    'serializer' => Redis::SERIALIZER_MSGPACK,
    'compression' => Redis::COMPRESSION_LZ4,
],
```

## Interacting With Redis

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');
$values = Redis::lrange('names', 5, 10);
```

### Using Multiple Connections

```php
$redis = Redis::connection('connection-name');
$redis = Redis::connection(); // default
```

### Transactions

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

### Lua Scripts

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

### Pipelining

```php
Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

## Pub/Sub

### Subscribing

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    protected $signature = 'redis:subscribe';

    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

### Publishing

```php
Redis::publish('test-channel', json_encode([
    'name' => 'Adam Wathan'
]));
```

### Wildcard Subscriptions

```php
Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/redis)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
