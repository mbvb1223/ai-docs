# Laravel 12.x Envoy Documentation

## Introduction

Laravel Envoy is a tool for executing common tasks on remote servers using Blade-style syntax. It supports deployment, Artisan commands, and more.

## Installation

```bash
composer require laravel/envoy --dev
```

## Writing Tasks

Create `Envoy.blade.php` in project root:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

### Local Tasks

```blade
@servers(['localhost' => '127.0.0.1'])
```

### Multiple Servers

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

### Parallel Execution

```blade
@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
@endtask
```

### Setup

```blade
@setup
    $now = new DateTime;
@endsetup
```

### Variables

```bash
php vendor/bin/envoy run deploy --branch=master
```

```blade
@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

## Stories

Group tasks together:

```blade
@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

## Completion Hooks

### @before

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

### @after

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

### @error

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

### @success

```blade
@success
    // All tasks succeeded
@endsuccess
```

### @finished

```blade
@finished
    if ($exitCode > 0) {
        // There were errors
    }
@endfinished
```

## Running Tasks

```bash
php vendor/bin/envoy run deploy
```

### Confirmation

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    // ...
@endtask
```

## Notifications

### Slack

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

### Discord

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

### Telegram

```blade
@finished
    @telegram('bot-id', 'chat-id')
@endfinished
```

### Microsoft Teams

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/envoy)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
