# Laravel 12.x Notifications Documentation

## Introduction

Laravel provides support for sending notifications across multiple delivery channels:
- Email
- SMS (via Vonage)
- Slack
- Database storage
- Broadcasting

## Generating Notifications

```bash
php artisan make:notification InvoicePaid
```

## Sending Notifications

### Using the Notifiable Trait

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

### Using the Notification Facade

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));

// Send immediately without queueing
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

### Specifying Delivery Channels

```php
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

### Queueing Notifications

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;
}
```

#### Delaying Notifications

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->plus(minutes: 5),
    'sms' => now()->plus(minutes: 10),
]));
```

### On-Demand Notifications

```php
Notification::route('mail', '[email protected]')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->notify(new InvoicePaid($invoice));
```

## Mail Notifications

### Formatting Mail Messages

```php
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->line('One of your invoices has been paid!')
        ->action('View Invoice', $url)
        ->line('Thank you for using our application!');
}
```

#### Error Messages

```php
return (new MailMessage)
    ->error()
    ->subject('Invoice Payment Failed')
    ->line('...');
```

### Customizing the Sender

```php
return (new MailMessage)
    ->from('[email protected]', 'Barrett Blair')
    ->line('...');
```

### Customizing the Recipient

```php
public function routeNotificationForMail(Notification $notification): array|string
{
    return [$this->email_address => $this->name];
}
```

### Mail Attachments

```php
return (new MailMessage)
    ->greeting('Hello!')
    ->attach('/path/to/file', [
        'as' => 'name.pdf',
        'mime' => 'application/pdf',
    ]);
```

### Tags and Metadata

```php
return (new MailMessage)
    ->greeting('Comment Upvoted!')
    ->tag('upvote')
    ->metadata('comment_id', $this->comment->id);
```

### Using Mailables

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email);
}
```

## Markdown Mail Notifications

### Generating

```bash
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

### Writing the Message

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

## Database Notifications

### Prerequisites

```bash
php artisan make:notifications-table
php artisan migrate
```

### Formatting

```php
public function toArray(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

### Accessing Notifications

```php
foreach ($user->notifications as $notification) {
    echo $notification->type;
}

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

### Marking as Read

```php
$notification->markAsRead();
$user->unreadNotifications->markAsRead();
$user->unreadNotifications()->update(['read_at' => now()]);
```

## Broadcast Notifications

### Formatting

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

### Listening for Notifications

```javascript
Echo.private('App.Models.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

## SMS Notifications (Vonage)

### Prerequisites

```bash
composer require laravel/vonage-notification-channel guzzlehttp/guzzle
```

### Formatting

```php
use Illuminate\Notifications\Messages\VonageMessage;

public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content');
}
```

### Routing

```php
public function routeNotificationForVonage(Notification $notification): string
{
    return $this->phone_number;
}
```

## Slack Notifications

### Prerequisites

```bash
composer require laravel/slack-notification-channel
```

### Formatting with Block Kit

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
            $block->field("*Invoice No:*\n1000")->markdown();
        });
}
```

### Interactive Buttons

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;

->actionsBlock(function (ActionsBlock $block) {
    $block->button('Acknowledge Invoice')->primary();
    $block->button('Deny')->danger()->id('deny_invoice');
});
```

### Routing

```php
public function routeNotificationForSlack(Notification $notification): mixed
{
    return '#support-channel';
}
```

## Testing

```php
use Illuminate\Support\Facades\Notification;

Notification::fake();

$user->notify(new InvoicePaid($invoice));

Notification::assertSentTo($user, InvoicePaid::class);
Notification::assertNotSentTo($user, AnotherNotification::class);
Notification::assertNothingSent();
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/notifications)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
