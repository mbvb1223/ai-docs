# Primitive Utilities

## Overview

Tempest provides utilities that streamline work with primitive values. The framework offers both functional and object-oriented APIs for handling strings and arrays, plus namespaced functions for regex operations, math, filesystem tasks, path handling, JSON manipulation, random generation, pluralization, and PHP namespaces.

## Namespaced Functions

The `Tempest\Support` namespace contains function-based implementations across several categories:

- **Regular expressions** — Pattern matching and manipulation
- **Arithmetic operations** — Mathematical functions
- **Filesystem operations** — File I/O tasks
- **Filesystem paths** — Path utilities
- **Json manipulation** — JSON handling
- **Random values** — Generation of random data
- **Pluralization** — Singular/plural conversion
- **PHP namespaces** — Namespace utilities

The framework also provides `IsEnumHelper` trait for enumeration handling.

## String Utilities

Developers can work with strings via functional API or fluent object-oriented approach. Two classes support this:

- `Tempest\Support\Str\ImmutableString`
- `Tempest\Support\Str\MutableString`

**Example with object-oriented API:**

```php
use Tempest\Support\Str\ImmutableString;

$slug = new ImmutableString('/blog/01-chasing-bugs-down-the-rabbit-hole/')
    ->stripEnd('/')
    ->afterLast('/')
    ->replaceRegex('/\d+-/', '')
    ->slug()
    ->toString();
```

The `str()` function provides shorthand for creating `ImmutableString` instances.

## Array Utilities

Array handling similarly supports both functional and object-oriented patterns via:

- `Tempest\Support\Arr\ImmutableArray`
- `Tempest\Support\Arr\MutableArray`

**Example:**

```php
use Tempest\Support\Arr\ImmutableArray;

$items = new ImmutableArray(glob(__DIR__ . '/content/*.md'))
    ->reverse()
    ->map(function (string $path) {
        // …
    })
    ->mapTo(BlogPost::class);
```

The `arr()` function creates `ImmutableArray` instances as a convenience.

## Recommendations

The documentation suggests preferring primitive utilities over PHP's built-in methods. For example, use `Filesystem\read_file` instead of native file functions—it handles edge cases better and throws clearer exceptions.

Both approaches work for simple operations:

```php
// Object-oriented
$title = str('My title')->title()->toString();

// Functional
$title = Str\to_title_case('My title');
```
