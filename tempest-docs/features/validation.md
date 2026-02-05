# Validation

## Overview

Tempest provides a `Validator` object for validating arrays of values against class properties or custom rule arrays. Validation and data mapping often work together, though the two are separate components and can also be used separately.

## Validating Against Objects

Use `validateValuesForClass()` to validate raw data against a model or data transfer object:

```php
$failingRules = $this->validator->validateValuesForClass(Book::class, [
    'title' => 'Timeline Taxi',
    'description' => 'My sci-fi novel',
    'publishedAt' => '2024-10-01',
]);
```

The validator infers rules from built-in PHP types on the target class properties.

### Adding More Rules

Enhance validation with attributes when PHP types alone are insufficient:

```php
use Tempest\Validation\Rules;

final class Book
{
    #[Rules\HasLength(min: 5, max: 50)]
    public string $title;

    #[Rules\IsNotEmptyString]
    public string $description;

    #[Rules\HasDateTimeFormat('Y-m-d')]
    public ?DateTime $publishedAt = null;
}
```

### Skipping Validation

Prevent specific properties from validation using `#[SkipValidation]`:

```php
use Tempest\Validation\SkipValidation;

final class Book
{
    #[SkipValidation]
    public string $title;
}
```

## Validating Against Specific Rules

Validate data without a model by providing custom rule arrays:

```php
$this->validator->validateValues([
    'name' => 'Jon Doe',
    'email' => 'jon@doe.co',
    'age' => 25,
], [
    'name' => [new IsString(), new IsNotNull()],
    'email' => [new IsEmail()],
    'age' => [new IsInteger(), new IsNotNull()],
]);
```

## Validating a Single Value

Validate individual values using `validateValue()`:

```php
$this->validator->validateValue('jon@doe.co', [new IsEmail()]);
```

Support for closure-based validation:

```php
$this->validator->validateValue('jon@doe.co', function (mixed $value) {
    return str_contains($value, '@');
});
```

## Accessing Error Messages

Retrieve localized error messages for failed validations:

```php
use Tempest\Support\Arr;
use Tempest\Validation\Rules\IsEmail;

$failures = $this->validator->validateValue('jon@doe.co', new IsEmail());
$errors = Arr\map_iterable($failures,
    fn (FailingRule $failure) => $this->validator->getErrorMessage($failure)
);
```

Specify field names for context-aware messages:

```php
$this->validator->getErrorMessage($failure, 'email');
// => 'Email must be a valid email address'
```

## Overriding Translation Messages

Customize validation messages via translation files:

```yaml
# app/Localization/validation.en.yml
validation_error:
  is_email: |
    .input {$field :string}
    {$field} must be a valid email address.
```

Use `#[TranslationKey]` for field-specific error messages:

```php
final class Book {
    #[Rules\HasLength(min: 5, max: 50)]
    #[TranslationKey('book_management.book_title')]
    public string $title;
}
```
