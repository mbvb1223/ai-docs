# Laravel 12.x Dusk Browser Testing Documentation

## Introduction

Laravel Dusk provides an expressive, easy-to-use browser automation and testing API using ChromeDriver.

## Installation

```bash
composer require laravel/dusk --dev
php artisan dusk:install
```

### Managing ChromeDriver

```bash
php artisan dusk:chrome-driver
php artisan dusk:chrome-driver 86
php artisan dusk:chrome-driver --detect
```

## Getting Started

### Generating Tests

```bash
php artisan dusk:make LoginTest
```

### Running Tests

```bash
php artisan dusk
php artisan dusk:fails
php artisan dusk --group=foo
```

### Database Reset

```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);
```

## Browser Basics

### Creating Browsers

```php
use App\Models\User;
use Laravel\Dusk\Browser;

test('basic example', function () {
    $user = User::factory()->create();

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

### Multiple Browsers

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))->visit('/home');
    $second->loginAs(User::find(2))->visit('/home');
});
```

### Navigation

```php
$browser->visit('/login');
$browser->visitRoute($routeName, $parameters);
$browser->back();
$browser->forward();
$browser->refresh();
```

### Authentication

```php
$browser->loginAs(User::find(1))->visit('/home');
```

### Screenshots

```php
$browser->screenshot('filename');
$browser->responsiveScreenshots('filename');
```

## Interacting With Elements

### Dusk Selectors

```html
<button dusk="login-button">Login</button>
```

```php
$browser->click('@login-button');
```

### Typing Values

```php
$browser->type('email', '[email protected]');
$browser->append('tags', ', bar, baz');
$browser->clear('email');
$browser->typeSlowly('mobile', '+1 (202) 555-5555');
```

### Dropdowns and Checkboxes

```php
$browser->select('size', 'Large');
$browser->check('terms');
$browser->uncheck('terms');
$browser->radio('size', 'large');
```

### File Uploads

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

### Clicking

```php
$browser->click('.selector');
$browser->doubleClick('.selector');
$browser->rightClick('.selector');
$browser->clickLink($linkText);
```

### Using the Keyboard

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
```

### Using the Mouse

```php
$browser->mouseover('.selector');
$browser->drag('.from-selector', '.to-selector');
```

## Waiting for Elements

```php
$browser->waitFor('.selector');
$browser->waitFor('.selector', 1);
$browser->waitUntilMissing('.selector');
$browser->waitForText('Hello World');
$browser->waitForLink('Create');
$browser->waitForLocation('/secret');
```

### Scoping When Available

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')->press('OK');
});
```

## Available Assertions

### URL Assertions

```php
$browser->assertTitle($title);
$browser->assertUrlIs($url);
$browser->assertPathIs('/home');
$browser->assertRouteIs($name, $parameters);
$browser->assertQueryStringHas($name, $value);
```

### Content Assertions

```php
$browser->assertSee($text);
$browser->assertDontSee($text);
$browser->assertSeeIn($selector, $text);
$browser->assertSeeLink($linkText);
```

### Form Assertions

```php
$browser->assertInputValue($field, $value);
$browser->assertChecked($field);
$browser->assertNotChecked($field);
$browser->assertSelected($field, $value);
```

### Element Assertions

```php
$browser->assertVisible($selector);
$browser->assertPresent($selector);
$browser->assertMissing($selector);
$browser->assertEnabled($field);
$browser->assertDisabled($field);
$browser->assertFocused($field);
```

### Authentication Assertions

```php
$browser->assertAuthenticated();
$browser->assertGuest();
$browser->assertAuthenticatedAs($user);
```

## Pages

### Generating Pages

```bash
php artisan dusk:page Login
```

### Configuring Pages

```php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Login extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
            '@password' => 'input[name=password]',
        ];
    }
}
```

### Using Pages

```php
use Tests\Browser\Pages\Login;

$browser->visit(new Login);
```

## Components

### Generating Components

```bash
php artisan dusk:component DatePicker
```

### Using Components

```php
use Tests\Browser\Components\DatePicker;

$browser->visit('/')
    ->within(new DatePicker, function ($browser) {
        $browser->assertSee('January');
    });
```

## Continuous Integration

### GitHub Actions

```yaml
name: Dusk

on: [push]

jobs:
  browser-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: php-actions/composer@v6
      - run: php artisan dusk:chrome-driver
      - run: npm install && npm run build
      - run: php artisan serve &
      - run: php artisan dusk
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/dusk)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
