# Laravel 12.x Valet Documentation

## Introduction

Laravel Valet is a development environment for macOS that configures Nginx to run in the background, using DnsMasq to proxy requests on the `*.test` domain.

## Installation

```bash
brew update
brew install php
composer global require laravel/valet
valet install
```

Verify:
```bash
ping foobar.test
```

## PHP Version Management

```bash
valet use [email protected]
valet use php  # revert to default
```

Create `.valetrc` in project root:
```
php=php@8.2
```

## Serving Sites

### Park Command

Register a directory containing multiple apps:

```bash
cd ~/Sites
valet park
```

Access at `http://<directory-name>.test`

### Link Command

Serve a single application:

```bash
cd ~/Sites/laravel
valet link
```

Access at `http://laravel.test`

Custom hostname:

```bash
valet link application
# http://application.test
```

List links:
```bash
valet links
```

Remove link:
```bash
valet unlink
```

## Securing Sites (TLS)

```bash
valet secure laravel
valet unsecure laravel
```

## Per-Site PHP Versions

```bash
cd ~/Sites/example-site
valet isolate [email protected]
```

List isolated sites:
```bash
valet isolated
```

Revert:
```bash
valet unisolate
```

## Sharing Sites

Choose sharing tool:

```bash
valet share-tool ngrok
valet share-tool expose
valet share-tool cloudflared
```

Share:

```bash
valet share
```

With ngrok token:
```bash
valet set-ngrok-token YOUR_TOKEN_HERE
```

## Environment Variables

Create `.valet-env.php` in project root:

```php
<?php

return [
    'laravel' => [
        'key' => 'value',
    ],
    '*' => [
        'key' => 'value',
    ],
];
```

## Proxying Services

```bash
valet proxy elasticsearch http://127.0.0.1:9200
valet proxy elasticsearch http://127.0.0.1:9200 --secure
valet unproxy elasticsearch
valet proxies
```

## Other Commands

| Command | Description |
|---------|-------------|
| `valet list` | Display all commands |
| `valet diagnose` | Output diagnostics |
| `valet forget` | Remove parked directory |
| `valet log` | View logs |
| `valet paths` | View parked paths |
| `valet restart` | Restart daemons |
| `valet start` | Start daemons |
| `valet stop` | Stop daemons |
| `valet trust` | Add sudoers files |
| `valet uninstall` | Uninstall Valet |

## Custom Valet Drivers

Place in `~/.config/valet/Drivers/`:

```php
class CustomValetDriver extends \Valet\Drivers\BasicValetDriver
{
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return is_dir($sitePath.'/wp-admin');
    }

    public function isStaticFile(string $sitePath, string $siteName, string $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }
        return false;
    }

    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public/index.php';
    }
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/valet)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
