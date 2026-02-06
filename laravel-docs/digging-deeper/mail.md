# Laravel 12.x Mail Documentation

## Introduction

Laravel provides a clean, simple email API powered by the Symfony Mailer component. It supports sending email via SMTP, Mailgun, Postmark, Resend, Amazon SES, and `sendmail`.

## Configuration

Configure email services via `config/mail.php`. The `mailers` array contains configurations for major mail drivers, with `default` specifying the default mailer.

### Driver Prerequisites

#### Mailgun Driver

```bash
composer require symfony/mailgun-mailer symfony/http-client
```

Configure in `config/services.php`:
```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

#### Postmark Driver

```bash
composer require symfony/postmark-mailer symfony/http-client
```

Configure in `config/services.php`:
```php
'postmark' => [
    'key' => env('POSTMARK_API_KEY'),
],
```

#### Resend Driver

```bash
composer require resend/resend-php
```

Configure in `config/services.php`:
```php
'resend' => [
    'key' => env('RESEND_API_KEY'),
],
```

#### SES Driver

```bash
composer require aws/aws-sdk-php
```

Configure in `config/services.php`:
```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

### Failover Configuration

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],
],
```

### Round Robin Configuration

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],
],
```

## Generating Mailables

```bash
php artisan make:mail OrderShipped
```

## Writing Mailables

### Configuring the Sender

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('[email protected]', 'Jeffrey Way'),
        replyTo: [
            new Address('[email protected]', 'Taylor Otwell'),
        ],
        subject: 'Order Shipped',
    );
}
```

### Configuring the View

```php
use Illuminate\Mail\Mailables\Content;

public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

#### Plain Text Emails

```php
public function content(): Content
{
    return new Content(
        html: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

### View Data

#### Via Public Properties

```php
class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

#### Via the `with` Parameter

```php
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        with: [
            'orderName' => $this->order->name,
            'orderPrice' => $this->order->price,
        ],
    );
}
```

### Attachments

```php
use Illuminate\Mail\Mailables\Attachment;

public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

#### From Storage

```php
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
        Attachment::fromStorageDisk('s3', '/path/to/file'),
    ];
}
```

#### Raw Data Attachments

```php
public function attachments(): array
{
    return [
        Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
            ->withMime('application/pdf'),
    ];
}
```

### Inline Attachments

```blade
<body>
    Here is an image:
    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

### Headers

```php
use Illuminate\Mail\Mailables\Headers;

public function headers(): Headers
{
    return new Headers(
        messageId: '[email protected]',
        references: ['[email protected]'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

### Tags and Metadata

```php
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

## Markdown Mailables

### Generating

```bash
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

### Writing Markdown Messages

```blade
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

#### Components

**Button:**
```blade
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

**Panel:**
```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

**Table:**
```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

### Customizing Components

```bash
php artisan vendor:publish --tag=laravel-mail
```

## Sending Mail

### Basic Usage

```php
use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

Mail::to($request->user())->send(new OrderShipped($order));
```

### Setting Recipients

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

### Using a Specific Mailer

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

## Queueing Mail

### Basic Queueing

```php
Mail::to($request->user())
    ->queue(new OrderShipped($order));
```

### Delayed Queueing

```php
Mail::to($request->user())
    ->later(now()->plus(minutes: 10), new OrderShipped($order));
```

### Pushing to Specific Queues

```php
$message = (new OrderShipped($order))
    ->onConnection('sqs')
    ->onQueue('emails');

Mail::to($request->user())->queue($message);
```

### Queueing by Default

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}
```

## Rendering Mailables

### As HTML

```php
return (new InvoicePaid($invoice))->render();
```

### Preview in Browser

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);
    return new App\Mail\InvoicePaid($invoice);
});
```

## Localizing Mailables

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

### User Preferred Locales

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    public function preferredLocale(): string
    {
        return $this->locale;
    }
}
```

## Testing Mailables

### Testing Content

```php
$mailable = new InvoicePaid($user);

$mailable->assertFrom('[email protected]');
$mailable->assertTo('[email protected]');
$mailable->assertHasSubject('Invoice Paid');
$mailable->assertSeeInHtml($user->email);
$mailable->assertSeeInText($user->email);
$mailable->assertHasAttachment('/path/to/file');
```

### Testing Sending

```php
use Illuminate\Support\Facades\Mail;

Mail::fake();

// Send mail...

Mail::assertSent(OrderShipped::class);
Mail::assertSent(OrderShipped::class, 2);
Mail::assertNotSent(AnotherMailable::class);
Mail::assertQueued(OrderShipped::class);
Mail::assertNothingSent();
```

## Events

- `MessageSending` - before sending
- `MessageSent` - after sending

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/mail)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
