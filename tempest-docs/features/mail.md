# Mail

## Overview

Tempest provides a convenient layer built on top of Symfony's excellent mailer component so that you can send emails with ease.

## Getting Started

### Configuration

The framework uses SMTP by default. Configure these environment variables:

```
MAIL_SMTP_HOST=mail.my_provider.com
MAIL_SMTP_PORT=587
MAIL_SMTP_USERNAME=my_username@my_provider.com
MAIL_SMTP_PASSWORD=my_password_123
MAIL_SENDER_NAME=Brent
MAIL_SENDER_EMAIL=brendt@stitcher.io
```

### Basic Usage

Inject the `Mailer` class to send emails:

```php
use Tempest\Mail\Mailer;
use Tempest\Mail\GenericEmail;

final class UserEventHandlers
{
    public function __construct(
        private readonly Mailer $mailer,
    ) {}

    #[EventHandler]
    public function onCreated(UserCreated $userCreated): void
    {
        $this->mailer->send(new GenericEmail(
            subject: 'Welcome!',
            to: $userCreated->email,
            html: view(
                __DIR__ . '/mails/welcome.view.php',
                user: $userCreated->user,
            ),
        ));
    }
}
```

### Custom Email Classes

For scalability, create individual email classes:

```php
use Tempest\Mail\Email;
use Tempest\Mail\Envelope;
use Tempest\View\View;

final class WelcomeEmail implements Email
{
    public function __construct(
        private readonly User $user,
    ) {}

    public Envelope $envelope {
        get => new Envelope(
            subject: 'Welcome',
            to: $this->user->email,
        );
    }

    public string|View $html {
        get => view('welcome.view.php', user: $this->user);
    }
}
```

## Email Content

### HTML Content

Pass HTML directly or use views:

```php
public string|View $html {
    get => <<<HTML
    <h1>Thanks for joining!</h1>
    HTML;
}
```

### Text-Only Content

Implement `HasTextContent` interface for text versions:

```php
use Tempest\Mail\HasTextContent;

final class WelcomeEmail implements Email, HasTextContent
{
    public string|View|null $text {
        get => view('welcome-text.view.php', user: $this->user);
    }
}
```

Text automatically strips HTML, but manual specification is recommended for better control.

## Attachments

Implement `HasAttachments` to add files:

```php
use Tempest\Mail\Attachment;
use Tempest\Mail\HasAttachments;

final class WelcomeEmail implements Email, HasAttachments
{
    public array $attachments {
        get => [
            Attachment::fromFilesystem(__DIR__ . '/welcome.pdf')
        ];
    }
}
```

### Attachment Methods

- **Filesystem**: `Attachment::fromFilesystem()`
- **Storage**: `Attachment::fromStorage($s3Storage, '/welcome.pdf')`
- **Dynamic**: Custom closure for generated content

## Transport Options

### Built-In Transports

Tempest supports SMTP, Amazon SES, and Postmark out of the box.

### Switching Transports (Postmark Example)

1. Install dependencies:
```
composer require symfony/postmark-mailer
composer require symfony/http-client
```

2. Create configuration file:

```php
use Tempest\Mail\Transports\Postmark\PostmarkConfig;

return new PostmarkConfig(
    key: env('MAIL_POSTMARK_TOKEN'),
    defaultSender: new EmailAddress(
        email: env('MAIL_SENDER_EMAIL'),
        name: env('MAIL_SENDER_NAME'),
    ),
);
```

### Custom Transports

For unsupported transports, create a config class implementing `MailerConfig`:

```php
final class MailgunConfig implements MailerConfig, ProvidesDefaultSender
{
    public function createTransport(): TransportInterface
    {
        return new MailgunTransportFactory()
            ->create(Dsn::fromString(
                "mailgun+api://{$this->key}:{$this->domain}@default"
            ));
    }
}
```

## Testing

Use `MailTester` in test classes extending `IntegrationTest`:

```php
public function test_welcome_mail()
{
    $this->mailer
        ->send(new WelcomeEmail($this->user))
        ->assertSentTo($this->user->email)
        ->assertAttached('welcome.pdf');
}
```

Emails sent during testing are intercepted and never actually sent.
