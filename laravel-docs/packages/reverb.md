# Laravel 12.x Reverb Documentation

## Introduction

Laravel Reverb provides blazing-fast, scalable real-time WebSocket communication with seamless integration with Laravel's event broadcasting.

## Installation

```bash
php artisan install:broadcasting
```

## Configuration

### Application Credentials

```env
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

### Allowed Origins

```php
'apps' => [
    [
        'app_id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
    ]
]
```

### SSL Configuration

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

## Running the Server

```bash
php artisan reverb:start
```

### Custom Host and Port

```bash
php artisan reverb:start --host=127.0.0.1 --port=9000
```

### Debugging

```bash
php artisan reverb:start --debug
```

### Restarting

```bash
php artisan reverb:restart
```

## Monitoring with Pulse

```php
'recorders' => [
    ReverbConnections::class => ['sample_rate' => 1],
    ReverbMessages::class => ['sample_rate' => 1],
],
```

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
</x-pulse>
```

## Production Deployment

### Nginx Configuration

```nginx
location / {
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_pass http://0.0.0.0:8080;
}
```

### Horizontal Scaling

```env
REVERB_SCALING_ENABLED=true
```

## Events

- `ChannelCreated` - First subscription to channel
- `ChannelRemoved` - Last unsubscribe from channel
- `ConnectionPruned` - Stale connection pruned
- `MessageReceived` - Message received
- `MessageSent` - Message sent

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/reverb)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
