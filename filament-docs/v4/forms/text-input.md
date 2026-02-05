# Filament Text Input Documentation

## Overview

The text input component in Filament allows interaction with string values in forms. Here's the basic implementation:

```php
use Filament\Forms\Components\TextInput;

TextInput::make('name')
```

## HTML Input Types

Filament provides convenience methods for common input types, many with built-in validation:

```php
TextInput::make('text')
    ->email()
    ->numeric()
    ->integer()
    ->password()
    ->tel()
    ->url()
```

You can also use the generic `type()` method for other HTML input types:

```php
TextInput::make('backgroundColor')
    ->type('color')
```

## Input Mode & Step Configuration

Set the [`inputmode` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#inputmode):

```php
TextInput::make('text')
    ->numeric()
    ->inputMode('decimal')
```

For numeric inputs, configure the step value:

```php
TextInput::make('number')
    ->numeric()
    ->step(100)
```

## Autocomplete Features

### Browser Autocomplete

```php
TextInput::make('password')
    ->password()
    ->autocomplete('new-password')
    // or disable with:
    ->autocomplete(false)
```

### Datalist Suggestions

Provide autocomplete recommendations (users can still enter any value):

```php
TextInput::make('manufacturer')
    ->datalist([
        'BMW',
        'Ford',
        'Mercedes-Benz',
        'Porsche',
        'Toyota',
        'Volkswagen',
    ])
```

## Autocapitalization

```php
TextInput::make('name')
    ->autocapitalize('words')
```

## Prefix & Suffix

Add text or icons before/after the input:

```php
TextInput::make('domain')
    ->prefix('https://')
    ->suffix('.com')

// With icons:
TextInput::make('domain')
    ->url()
    ->suffixIcon(Heroicon::GlobeAlt)
    ->suffixIconColor('success')
```

## Password Visibility Toggle

```php
TextInput::make('password')
    ->password()
    ->revealable()
```

## Clipboard Copy Feature

```php
TextInput::make('apiKey')
    ->label('API key')
    ->copyable(copyMessage: 'Copied!', copyMessageDuration: 1500)
```

**Note:** SSL must be enabled for this feature.

## Input Masking

Apply format constraints using Alpine.js masks:

```php
TextInput::make('birthday')
    ->mask('99/99/9999')
    ->placeholder('MM/DD/YYYY')
```

For dynamic masks:

```php
TextInput::make('cardNumber')
    ->mask(RawJs::make(<<<'JS'
        $input.startsWith('34') || $input.startsWith('37') ? '9999 999999 99999' : '9999 9999 9999 9999'
    JS))
```

Strip masked characters before validation:

```php
TextInput::make('amount')
    ->mask(RawJs::make('$money($input)'))
    ->stripCharacters(',')
    ->numeric()
```

## Whitespace Trimming

```php
TextInput::make('name')
    ->trim()
```

Enable globally via service provider:

```php
TextInput::configureUsing(function (TextInput $component): void {
    $component->trim();
});
```

## Read-Only Mode

```php
TextInput::make('name')
    ->readOnly()
```

Differs from `disabled()`: field still submits, no styling changes, remains focusable.

## Validation

### Length Validation

```php
TextInput::make('name')
    ->minLength(2)
    ->maxLength(255)
    ->length(8)  // exact length
```

### Size Validation (Numeric)

```php
TextInput::make('number')
    ->numeric()
    ->minValue(1)
    ->maxValue(100)
```

### Phone Number Validation

Default regex: `/^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\.\/0-9]*$/`

```php
TextInput::make('phone')
    ->tel()
    ->telRegex('/^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\.\/0-9]*$/')
```

Customize globally:

```php
TextInput::configureUsing(function (TextInput $component): void {
    $component->telRegex('/your-custom-regex/');
});
```

---

Most methods support dynamic calculation via callback functions with utility injection (accessing `$get`, `$state`, `$record`, etc.).
