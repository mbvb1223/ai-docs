# Localization

## Overview

Tempest provides localization utilities featuring a translator built on the MessageFormat 2.0 specification. The `Translator` interface enables message translation into different languages with locale-specific formatting. The implementation follows standards maintained by the Unicode project and is widely used in internationalization libraries.

## Translating Messages

Inject the `Tempest\Intl\Translator` interface and use its `translate()` method. Pass variables as named parameters:

```php
$translator->translate('cart.expire_at', expire_at: $expiration);
// Your cart is valid until 1:30 PM
```

For specific locales, use `translateForLocale()`:

```php
$translator->translateForLocale(Locale::FRENCH, 'cart.expire_at', expire_at: $expiration);
// Votre panier expire à 12h30
```

Alternatively, use helper functions from the `Tempest\Intl` namespace: `translate()` or `translate_for_locale()`.

## Configuring the Locale

The current locale is stored in the `currentLocale` property of `IntlConfig`. Create a configuration file to set defaults:

```php
// intl.config.php
return new IntlConfig(
    currentLocale: Locale::FRENCH,
    fallbackLocale: Locale::ENGLISH,
);
```

By default, Tempest uses the PHP `intl.default_locale` ini value.

## Changing the Locale

Update the locale at runtime by mutating the `IntlConfig` object, such as in middleware:

```php
final readonly class SetLocaleMiddleware implements HttpMiddleware
{
    public function __construct(
        private Authenticator $authenticator,
        private IntlConfig $intlConfig,
    ) {}

    public function __invoke(Request $request, HttpMiddlewareCallable $next): Response
    {
        $this->intlConfig->currentLocale = $this->authenticator
            ->currentUser()
            ->preferredLocale;

        return $next($request);
    }
}
```

## Defining Translation Messages

Tempest auto-discovers YAML and JSON translation files using the `<name>.<locale>.{yaml,json}` naming pattern, where `<locale>` is an ISO 639-1 language code.

Example directory structure:

```
src/
└── lang/
    ├── messages.fr.yaml
    └── messages.en.yaml
```

Add messages at runtime using the `Catalog` class:

```php
$catalog->add(Locale::FRENCH, 'order.continue_shopping', 'Continuer vos achats');
```

## Message Syntax

Tempest implements MessageFormat 2.0, supporting variables, pluralization, and custom formatting functions. Example YAML message:

```yaml
today:
  Today is {$today :datetime pattern=|yyyy/MM/dd|}
```

Refer to the [MessageFormat documentation](https://messageformat.unicode.org/docs/translators/) for complete syntax details.

## Pluralization

Use matchers and the `number` function for plural support across multiple plural categories:

```yaml
cart:
  items_count:
    .input {$count :number}
    .match $count
      one   {{Masz {$count} przedmiot.}}
      few   {{Masz {$count} przedmioty.}}
      many  {{Masz {$count} przedmiotów.}}
      other {{Masz {$count} przedmiotów.}}
```

Support multiple variables in a single matcher for complex pluralization rules.

## Using Markup

Add markup using MessageFormat syntax. Tempest renders HTML tags and Iconify icons:

```yaml
bold_text: "This is {#strong}bold{/strong}."
ui:
  open_menu: "{#icon-tabler-menu/} Open menu"
```

Implement custom markup by creating classes that implement `MarkupFormatter` or `StandaloneMarkupFormatter` interfaces—Tempest auto-discovers them.

## Custom Formatting Functions

Define custom formatting functions by implementing the `FormattingFunction` interface. Default functions format strings, numbers, and dates.

Example datetime function implementation:

```php
final class DateTimeFunction implements FormattingFunction
{
    public string $name = 'datetime';

    public function format(mixed $value, array $parameters): FormattedValue
    {
        $datetime = DateTime::parse($value);
        $formatted = $datetime->format(Arr\get_by_key($parameters, 'pattern'));

        return new FormattedValue($value, $formatted);
    }
}
```
