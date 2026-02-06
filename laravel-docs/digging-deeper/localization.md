# Laravel 12.x Localization Documentation

## Introduction

Laravel's localization features provide a convenient way to retrieve strings in various languages.

## Publishing Language Files

```bash
php artisan lang:publish
```

### Language File Structure

**PHP files:**
```
/lang
    /en
        messages.php
    /es
        messages.php
```

**JSON files:**
```
/lang
    en.json
    es.json
```

## Configuring the Locale

### At Runtime

```php
use Illuminate\Support\Facades\App;

App::setLocale($locale);
```

### Determining Current Locale

```php
$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

### Pluralization Language

```php
use Illuminate\Support\Pluralizer;

Pluralizer::useLanguage('spanish');
```

## Defining Translation Strings

### Using Short Keys

```php
<?php
// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```

### Using Translation Strings as Keys (JSON)

```json
{
    "I love programming.": "Me encanta programar."
}
```

## Retrieving Translation Strings

```php
echo __('messages.welcome');
echo __('I love programming.');
```

In Blade:

```blade
{{ __('messages.welcome') }}
```

### Replacing Parameters

```php
// Definition
'welcome' => 'Welcome, :name',

// Usage
echo __('messages.welcome', ['name' => 'dayle']);
```

Capitalization:
```php
'welcome' => 'Welcome, :NAME',  // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name',  // Goodbye, Dayle
```

### Pluralization

```php
// Definition
'apples' => 'There is one apple|There are many apples',

// Complex rules
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

// Usage
echo trans_choice('messages.apples', 10);
```

With placeholders:

```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

## Overriding Package Language Files

Place files in:
```
lang/vendor/{package}/{locale}/
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/localization)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
