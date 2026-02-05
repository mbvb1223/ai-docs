# Configuration

## Overview

Tempest implements a distinctive configuration approach using objects, enabling code editors to deliver static analysis and autocomplete capabilities. The framework minimizes required configuration while allowing fine-grained control through available configuration classes when necessary.

## Configuration Files

Files matching the `*.config.php` pattern are automatically discovered and registered as singletons in the container.

Example PostgreSQL configuration:

```php
use Tempest\Database\Config\PostgresConfig;
use function Tempest\env;

return new PostgresConfig(
    host: env('DB_HOST'),
    port: env('DB_PORT'),
    username: env('DB_USERNAME'),
    password: env('DB_PASSWORD'),
    database: env('DB_DATABASE'),
);
```

## Accessing Configuration Objects

Inject configuration objects into controllers, services, or any container-resolvable class:

```php
use Tempest\Core\Environment;

final readonly class AboutController
{
    public function __construct(
        private Environment $environment,
    ) {}

    #[Get('/')]
    public function __invoke(): View
    {
        return view('about.view.php', environment: $this->environment);
    }
}
```

## Updating Configuration Objects

Modify configuration properties directly—changes persist as singletons throughout the application lifecycle:

```php
use Tempest\Support\Random;
use Tempest\Vite\ViteConfig;

$this->viteConfig->nonce = Random\secure_string(length: 40);
```

Alternatively, override the entire configuration instance:

```php
$this->container->config(new SQLiteConfig(
    path: root_path('database.sqlite'),
));
```

## Creating Custom Configuration

Define configuration classes with required properties and optional defaults:

```php
final class SlackConfig
{
    public function __construct(
        public string $token,
        public string $baseUrl,
        public string $applicationId,
        public ?string $userAgent = null,
    ) {}
}
```

Register via `slack.config.php`:

```php
use function Tempest\env;

return new SlackConfig(
    token: env('SLACK_API_TOKEN'),
    baseUrl: env('SLACK_BASE_URL', default: 'https://slack.com/api'),
    applicationId: env('SLACK_APP_ID'),
    userAgent: env('USER_AGENT'),
);
```

Inject into services or controllers as needed.

## Per-Environment Configuration

Use environment-specific suffixes for different deployment contexts:

- Production: `.prd.config.php`, `.prod.config.php`, `.production.config.php`
- Staging: `.stg.config.php`, `.staging.config.php`
- Development: `.dev.config.php`, `.local.config.php`
- Testing: `.test.config.php`, `.testing.config.php`

Example production storage configuration:

```php
return new S3StorageConfig(
    bucket: env('S3_BUCKET'),
    region: env('S3_REGION'),
    accessKeyId: env('S3_ACCESS_KEY_ID'),
    secretAccessKey: env('S3_SECRET_ACCESS_KEY'),
);
```

## Disabling Configuration Cache

During development, configuration files are discovered on each boot. In production, caching occurs automatically. Override this by setting the environment variable:

```
CONFIG_CACHE=true
```
