# Container

## Overview

A dependency container manages object creation and resolution within an application. Rather than manually instantiating dependencies, classes declare their requirements and the container provides them automatically. Tempest's container resolves dependencies without configuration, forming the foundation for controllers, console commands, event handlers, and the command bus.

## Injecting Dependencies

Class constructors resolved by the container can depend on any class or interface linked to a dependency initializer. Similarly, invoked methods like event handlers and console commands may be called directly from the container.

```php
use App\Aircraft\ExternalAircraftProvider;
use App\Aircraft\AircraftRepository;
use Tempest\Console\ConsoleCommand;

final readonly class AircraftService
{
    public function __construct(
        private ExternalAircraftProvider $externalAircraftProvider,
        private AircraftRepository $repository,
    ) {}

    #[ConsoleCommand]
    public function synchronize(): void
    {
        // …
    }
}
```

### Invoking Methods or Functions

Access the container instance to call the `invoke()` method, which resolves dependencies for other methods, functions, or invokable classes:

```php
$this->container->invoke(TrackOperatingAircraft::class, type: AircraftType::PC12);
```

The `\Tempest\invoke()` function provides the same functionality when the container isn't directly accessible.

### Locating a Dependency

When constructor injection isn't possible, use the `\Tempest\Container\get()` function:

```php
use function Tempest\Container\get;

$config = get(AppConfig::class);
```

**Note:** Service location should be a last resort. Refer to discussions on the service locator anti-pattern for context.

## Dependency Initializers

Initializers provide fine-grained control over dependency construction, replacing autowiring when needed. They're classes implementing the `Tempest\Container\Initializer` interface.

### Implementing an Initializer

Initializers implement `Initializer` with an `initialize()` method receiving the container and returning an instantiated object. The return type determines which dependency the initializer handles:

```php
use Tempest\Container\Container;
use Tempest\Container\Initializer;

final readonly class MarkdownInitializer implements Initializer
{
    public function initialize(Container $container): MarkdownConverter
    {
        $environment = new Environment();
        $highlighter = new Highlighter(new CssTheme());

        $highlighter
            ->addLanguage(new TempestViewLanguage())
            ->addLanguage(new TempestConsoleWebLanguage())
            ->addLanguage(new ExtendedJsonLanguage());

        $environment
            ->addExtension(new CommonMarkCoreExtension())
            ->addExtension(new FrontMatterExtension())
            ->addRenderer(FencedCode::class, new CodeBlockRenderer($highlighter))
            ->addRenderer(Code::class, new InlineCodeBlockRenderer($highlighter));

        return new MarkdownConverter($environment);
    }
}
```

### Matching Multiple Classes or Interfaces

Use union return types to match several classes to a single initializer:

```php
public function initialize(Container $container): MarkdownConverter|Markdown
{
    // …
}
```

### Dynamically Matching Classes or Interfaces

The `DynamicInitializer` interface provides flexible matching through a `canInitialize()` method:

```php
use Tempest\Container\Container;
use Tempest\Container\DynamicInitializer;
use Tempest\Reflection\ClassReflector;
use UnitEnum;

final class RouteBindingInitializer implements DynamicInitializer
{
    public function canInitialize(ClassReflector $class, null|string|UnitEnum $tag): bool
    {
        return $class->getType()->matches(Model::class);
    }

    public function initialize(ClassReflector $class, null|string|UnitEnum $tag, Container $container): object
    {
        // …
    }
}
```

## Autowired Dependencies

For simple one-to-one interface-to-implementation mappings, skip initializer boilerplate by using the `#[Autowire]` attribute on the implementation:

```php
use Tempest\Container\Autowire;

#[Autowire]
final readonly class AircraftService implements AircraftServiceInterface
{
    // …
}
```

Tempest discovers this and links the class to its interface automatically.

## Singletons

Register a class as a singleton using the `#[Singleton]` attribute:

```php
use Tempest\Container\Singleton;
use Tempest\HttpClient\HttpClient;

#[Singleton]
final readonly class Client
{
    public function __construct(
        private HttpClient $http,
    ) {}

    public function fetch(Icao $icao): Aircraft
    {
        // …
    }
}
```

Annotate initializer methods as `#[Singleton]` so their return objects are resolved only once:

```php
#[Singleton]
public function initialize(Container $container): MarkdownConverter|Markdown
{
    // …
}
```

### Tagged Singletons

Control singleton definitions with tags. Create multiple configured instances of the same class:

```php
use Tempest\Container\Container;
use Tempest\Container\Initializer;
use Tempest\Container\Singleton;

final readonly class WebHighlighterInitializer implements Initializer
{
    #[Singleton(tag: 'web')]
    public function initialize(Container $container): Highlighter
    {
        return new Highlighter(new CssTheme());
    }
}
```

Retrieve tagged instances during autowiring using the `#[Tag]` attribute:

```php
use Tempest\Container\Tag;

class HttpExceptionHandler implements ExceptionHandler
{
    public function __construct(
        #[Tag('web')]
        private Highlighter $highlighter,
    ) {}
}
```

Or directly from the container:

```php
$container->get(Highlighter::class, tag: 'cli');
```

### Dynamic Tags

Components implementing `Tempest\Container\HasTag` have a `tag` property that automatically tags singletons. This enables multiple instances with different configurations, supporting features like multiple database connections.

## Built-in Types Dependencies

Depend on built-in types like `string`, `int`, or `array` through tagged singletons (autowiring isn't available for these):

```php
use Tempest\Container\Container;
use Tempest\Container\Initializer;

final readonly class BookValidatorsInitializer implements Initializer
{
    #[Singleton(tag: 'book-validators')]
    public function initialize(Container $container): array
    {
        return [
            $container->get(HeaderValidator::class),
            $container->get(BodyValidator::class),
            $container->get(FooterValidator::class),
        ];
    }
}
```

Use the collection in your classes:

```php
use Tempest\Container\Tag;

final readonly class BookController
{
    public function __construct(
        #[Tag('book-validators')] private readonly array $contentValidators,
    ) { /* … */ }
}
```

## Injected Properties

Mark any property with the `#[Inject]` attribute to receive values directly when the container resolves a class instance:

```php
use Tempest\Container\Inject;

trait HasConsole
{
    #[Inject]
    private Console $console;

    // …
}
```

**Note:** While constructor injection is preferred, injected properties offer flexibility when using traits, avoiding constructor conflicts.

## Decorators

The decorator pattern wraps objects to add runtime behavior without structural changes. Useful for logging, caching, validation, and authentication.

To implement:
1. Apply the `#[Decorates]` attribute to your decorator class
2. Implement the interface of the decorated class
3. Accept the decorated object as a constructor parameter

```php
use Tempest\Container\Decorates;

#[Decorates(Repository::class)]
final readonly class CacheRepository implements Repository
{
    public function __construct(
        private Repository $repository,
        private Cache $cache,
    ) {}

    public function findById(int $id): ?Book
    {
        return $this->cache->resolve(
            key: "book.{$id}",
            callback: fn () => $this->repository->find($id)
        );
    }

    public function save(Book $book): Book
    {
        $this->cache->delete("book.{$book->id}");

        return $this->repository->save($book);
    }
}
```

Decorators are discovered automatically, so no manual registration is needed.

## Proxy Loading

Enable lazy loading of dependencies using the `#[Proxy]` attribute on properties with `#[Inject]` or constructor parameters. The container injects a transparent lazy proxy instead of the actual dependency—useful for heavy dependencies that may not be used:

```php
use Tempest\Container\Proxy;

final readonly class BookController
{
    public function __construct(
        #[Proxy]
        private VerySlowClass $verySlowClass
    ) { /* … */ }
}
```
