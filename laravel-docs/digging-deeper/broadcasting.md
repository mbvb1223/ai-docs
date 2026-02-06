# Laravel 12.x Broadcasting Documentation

## Introduction

WebSockets enable real-time, live-updating user interfaces. Laravel makes it easy to broadcast server-side events over WebSocket connections.

**Supported Drivers**: Laravel Reverb, Pusher Channels, Ably

## Server Side Installation

### Reverb

```bash
php artisan install:broadcasting --reverb
```

### Pusher Channels

```bash
php artisan install:broadcasting --pusher
composer require pusher/pusher-php-server
```

### Ably

```bash
php artisan install:broadcasting --ably
composer require ably/ably-php
```

## Client Side Installation

```bash
npm install --save-dev laravel-echo pusher-js
```

Configure in `resources/js/bootstrap.js`:

```javascript
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

## Defining Broadcast Events

```php
<?php

namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public function __construct(
        public User $user,
    ) {}

    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('user.'.$this->user->id),
        ];
    }
}
```

### Broadcast Name

```php
public function broadcastAs(): string
{
    return 'server.created';
}
```

### Broadcast Data

```php
public function broadcastWith(): array
{
    return ['id' => $this->user->id];
}
```

### Broadcast Queue

```php
public $connection = 'redis';
public $queue = 'default';
```

### Broadcast Conditions

```php
public function broadcastWhen(): bool
{
    return $this->order->value > 100;
}
```

## Authorizing Channels

In `routes/channels.php`:

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

### Channel Classes

```bash
php artisan make:channel OrderChannel
```

```php
Broadcast::channel('orders.{order}', OrderChannel::class);
```

## Broadcasting Events

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);

// Only to others
broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

### Anonymous Events

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

## Receiving Broadcasts

### Listening for Events

```javascript
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

### Private Channels

```javascript
Echo.private(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

### Stop Listening

```javascript
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated');
```

### Leaving Channels

```javascript
Echo.leaveChannel(`orders.${this.order.id}`);
```

### React Hooks

```javascript
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

## Presence Channels

### Authorizing

```php
Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

### Joining

```javascript
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // Users currently in channel
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```

## Model Broadcasting

```php
use Illuminate\Database\Eloquent\BroadcastsEvents;

class Post extends Model
{
    use BroadcastsEvents;

    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

### Listening for Model Broadcasts

```javascript
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.UserUpdated', (e) => {
        console.log(e.model);
    });
```

## Client Events

### Broadcasting

```javascript
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

### Listening

```javascript
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/broadcasting)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
