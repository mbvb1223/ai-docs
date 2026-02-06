# Laravel 12.x Strings Documentation

## Introduction

Laravel includes various functions for manipulating string values.

## String Helpers

### Basic Helpers

```php
// Translation
echo __('Welcome to our application');
echo __('messages.welcome');

// Class basename
$class = class_basename('Foo\Bar\Baz'); // 'Baz'

// HTML escape
echo e('<html>foo</html>'); // &lt;html&gt;foo&lt;/html&gt;
```

## Str Class Methods

### String Extraction

```php
use Illuminate\Support\Str;

Str::after('This is my name', 'This is');      // ' my name'
Str::afterLast('App\Http\Controllers', '\\');   // 'Controller'
Str::before('This is my name', 'my name');      // 'This is '
Str::beforeLast('This is my name', 'is');       // 'This '
Str::between('This is my name', 'This', 'name'); // ' is my '
```

### Case Conversion

```php
Str::camel('foo_bar');     // 'fooBar'
Str::kebab('fooBar');      // 'foo-bar'
Str::snake('fooBar');      // 'foo_bar'
Str::studly('foo_bar');    // 'FooBar'
Str::title('a nice title'); // 'A Nice Title'
Str::headline('steve_jobs'); // 'Steve Jobs'
Str::lower('LARAVEL');      // 'laravel'
Str::upper('laravel');      // 'LARAVEL'
Str::ucfirst('foo bar');    // 'Foo bar'
```

### String Checking

```php
Str::contains('This is my name', 'my');           // true
Str::containsAll('This is my name', ['my', 'name']); // true
Str::startsWith('This is my name', 'This');       // true
Str::endsWith('This is my name', 'name');         // true
Str::is('foo*', 'foobar');                        // true
Str::isJson('[1,2,3]');                           // true
Str::isUrl('http://example.com');                 // true
Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de'); // true
Str::isUlid('01gd6r360bp37zj17nxb55yv40');        // true
```

### String Replacement

```php
Str::replace('11.x', '12.x', 'Laravel 11.x');     // 'Laravel 12.x'
Str::replaceFirst('the', 'a', 'the quick fox');   // 'a quick fox'
Str::replaceLast('the', 'a', 'the fox the dog');  // 'the fox a dog'
Str::remove('e', 'Peter Piper');                  // 'Ptr Pipr'
Str::swap(['Tacos' => 'Burritos'], 'Tacos are great!'); // 'Burritos are great!'
```

### String Trimming

```php
Str::trim(' foo bar ');                // 'foo bar'
Str::squish('  laravel   framework  '); // 'laravel framework'
Str::deduplicate('The   Laravel', ' '); // 'The Laravel'
```

### String Manipulation

```php
Str::start('this/string', '/');        // '/this/string'
Str::finish('this/string', '/');       // 'this/string/'
Str::wrap('Laravel', '"');             // '"Laravel"'
Str::limit('The quick brown fox', 20); // 'The quick brown fox...'
Str::words('Perfectly balanced', 2);   // 'Perfectly balanced...'
Str::mask('[email protected]', '*', 3); // 'tay***************'
```

### String Generation

```php
Str::random(40);                       // Random 40 char string
Str::password();                       // Secure random password
Str::uuid();                           // UUID v4
Str::ulid();                           // ULID
Str::orderedUuid();                    // Timestamp-first UUID
Str::slug('Laravel 5 Framework');      // 'laravel-5-framework'
```

### String Encoding

```php
Str::ascii('û');                       // 'u'
Str::toBase64('Laravel');              // 'TGFyYXZlbA=='
Str::fromBase64('TGFyYXZlbA==');       // 'Laravel'
Str::markdown('# Laravel');            // '<h1>Laravel</h1>'
```

### Pluralization

```php
Str::plural('car');                    // 'cars'
Str::plural('child', 2);               // 'children'
Str::singular('cars');                 // 'car'
```

## Fluent Strings

```php
use Illuminate\Support\Str;

// Chain operations
$result = Str::of('foo bar')
    ->upper()
    ->replace('BAR', 'BAZ');
// 'FOO BAZ'

// Common operations
Str::of('This is my name')->after('This is');  // ' my name'
Str::of('foo_bar')->camel();                    // 'fooBar'
Str::of('This is my name')->contains('my');     // true
Str::of('  Laravel  ')->trim();                 // 'Laravel'

// Conditional operations
Str::of('Laravel')->when(true, fn ($str) => $str->append(' Framework'));
// 'Laravel Framework'

// Callbacks
Str::of('Hello World')->pipe(fn ($str) => strlen($str)); // 11
Str::of('foo bar baz')->split(' '); // collect(['foo', 'bar', 'baz'])

// Checks
Str::of('  ')->trim()->isEmpty();               // true
Str::of('Laravel')->exactly('Laravel');         // true
```

## str() Helper

```php
$string = str('Hello World');
$snake = str()->snake('FooBar'); // 'foo_bar'
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/strings)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
