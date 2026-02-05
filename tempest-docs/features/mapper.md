# Mapper

## Overview

The mapper component is capable of mapping data to objects and the other way around. It is one of Tempest's most powerful tools. The mapper handles request data transformation to request classes, SQL results to models, and other data conversions.

## Mapping Data

Use the `map()` function to transform source data into target objects:

```php
use function Tempest\Mapper\map;

$book = map($rawBookAsJson)->to(Book::class);
```

### Mapping to Collections

For array sources containing multiple items, apply the `collection()` method:

```php
use function Tempest\Mapper\map;

$books = map($rawBooksAsJson)
    ->collection()
    ->to(Book::class);
```

### Choosing Specific Mappers

Explicitly specify which mapper to use via the `with()` method:

```php
$psrRequest = map($request)
    ->with(RequestToPsrRequestMapper::class)
    ->do();
```

Alternatively, provide closures for advanced operations:

```php
$result = map($rawBooksAsJson)
    ->with(fn (ArrayToBooksMapper $mapper, array $books) =>
        $mapper->map($books, Book::class))
    ->do();
```

### Serializing to Arrays or JSON

Convert mapped objects using `toArray()` or `toJson()`:

```php
$array = map($book)->toArray();
$json = map($book)->toJson();
```

## Field Name Mapping

### MapFrom Attribute

Override source array keys using `#[MapFrom]`:

```php
use Tempest\Mapper\MapFrom;

final class Book
{
    #[MapFrom('book_title')]
    public string $title;
}

$book = map(['book_title' => 'Timeline Taxi'])->to(Book::class);
```

### MapTo Attribute

Specify serialization keys with `#[MapTo]`:

```php
use Tempest\Mapper\MapTo;

final class Book
{
    #[MapTo('book_title')]
    public string $title;
}
```

## Strict Mapping

By default, the mapper permits incomplete objects. Use the `#[Strict]` attribute to enforce all public properties:

```php
use Tempest\Mapper\Strict;
use function Tempest\Mapper\map;

#[Strict]
final class Book
{
    public string $title;
    public string $contents;
}

// Throws MappingValuesWereMissing
$book = map(['title' => 'Timeline Taxi'])->to(Book::class);
```

## Custom Mappers

Implement the `Tempest\Mapper\Mapper` interface:

```php
final readonly class PsrRequestToRequestMapper implements Mapper
{
    public function canMap(mixed $from, mixed $to): bool
    {
        if (! $from instanceof PsrRequest) {
            return false;
        }

        return is_a($to, Request::class, allow_string: true);
    }

    public function map(mixed $from, mixed $to): object
    { /* … */ }
}
```

Tempest automatically discovers classes implementing the `Mapper` interface.

## Casters and Serializers

**Casters** convert serialized data to complex types; **serializers** do the reverse.

### Basic Implementation

```php
use Tempest\Mapper\Caster;

final readonly class AddressCaster implements Caster
{
    public function cast(mixed $input): Address
    {
        return new Address(
            street: $input['street'],
            city: $input['city'],
            postalCode: $input['postal_code'],
        );
    }
}
```

```php
use Tempest\Mapper\Serializer;

final readonly class AddressSerializer implements Serializer
{
    public function serialize(mixed $input): array|string
    {
        if (! $input instanceof Address) {
            throw new ValueCouldNotBeSerialized(Address::class);
        }

        return $input->toArray();
    }
}
```

### Dynamic Registration

Implement `DynamicCaster` or `DynamicSerializer` for automatic global registration:

```php
use Tempest\Mapper\DynamicSerializer;

final readonly class AddressSerializer implements
    Serializer,
    DynamicSerializer
{
    public static function accepts(
        PropertyReflector|TypeReflector $input
    ): bool
    {
        $type = $input instanceof PropertyReflector
            ? $input->getType()
            : $input;

        return $type->matches(Address::class);
    }

    public function serialize(mixed $input): array|string
    {
        if (! $input instanceof Address) {
            throw new ValueCouldNotBeSerialized(Address::class);
        }

        return $input->toArray();
    }
}
```

### Property-Specific Casters/Serializers

Use attributes to apply specific casters to properties:

```php
use Tempest\Mapper\CastWith;
use Tempest\Mapper\SerializeWith;

final class User
{
    #[CastWith(AddressCaster::class)]
    #[SerializeWith(AddressSerializer::class)]
    public Address $address;
}
```

## Mapping Contexts

Contexts allow different serialization behavior based on situation (API vs. database):

### Specifying Context

```php
use App\SerializationContext;
use function Tempest\Mapper\map;

$json = map($book)
    ->in(SerializationContext::API)
    ->toJson();
```

### Context-Specific Serializers

Apply serializers only in specific contexts:

```php
use Tempest\Mapper\Attributes\Context;
use Tempest\Mapper\Serializer;
use Tempest\Mapper\DynamicSerializer;

#[Context(SerializationContext::API)]
final readonly class ApiDateSerializer
    implements Serializer, DynamicSerializer
{
    public static function accepts(
        PropertyReflector|TypeReflector $input
    ): bool
    {
        $type = $input instanceof PropertyReflector
            ? $input->getType()
            : $input;

        return $type->matches(DateTime::class);
    }

    public function serialize(mixed $input): string
    {
        return $input->format(FormatPattern::ISO8601);
    }
}
```

### Injecting Context

Inject context into serializer constructors:

```php
use Tempest\Mapper\Attributes\Context;
use Tempest\Mapper\Serializer;

#[Context(DatabaseContext::class)]
final class BooleanSerializer
    implements Serializer, DynamicSerializer
{
    public function __construct(
        private DatabaseContext $context,
    ) {}

    public static function accepts(
        PropertyReflector|TypeReflector $type
    ): bool
    {
        $type = $type instanceof PropertyReflector
            ? $type->getType()
            : $type;

        return $type->getName() === 'bool' ||
               $type->getName() === 'boolean';
    }

    public function serialize(mixed $input): string
    {
        return match ($this->context->dialect) {
            DatabaseDialect::POSTGRESQL =>
                $input ? 'true' : 'false',
            default => $input ? '1' : '0',
        };
    }
}
```

## Configurable Casters and Serializers

For property-dependent behavior, implement `ConfigurableCaster` or `ConfigurableSerializer`:

```php
use Tempest\Mapper\Caster;
use Tempest\Mapper\ConfigurableCaster;
use Tempest\Mapper\DynamicCaster;

final readonly class EnumCaster
    implements Caster, DynamicCaster, ConfigurableCaster
{
    /**
     * @param class-string<UnitEnum> $enum
     */
    public function __construct(
        private string $enum,
    ) {}

    public static function accepts(
        PropertyReflector|TypeReflector $input
    ): bool
    {
        $type = $input instanceof PropertyReflector
            ? $input->getType()
            : $input;

        return $type->matches(UnitEnum::class);
    }

    public static function configure(
        PropertyReflector $property,
        Context $context
    ): self
    {
        return new self(
            enum: $property->getType()->getName()
        );
    }

    public function cast(mixed $input): ?object
    {
        if ($input === null) {
            return null;
        }

        return $this->enum::from($input);
    }
}
```

Use configurable casters when behavior depends on specific property types, requires property metadata access, or different properties need different handling.
