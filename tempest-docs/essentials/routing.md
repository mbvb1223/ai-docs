# Routing

## Overview

Tempest uses PHP attributes on class methods to define routes. Routes can be attached to any class, though controllers are conventional. HTTP verb attributes like `Get`, `Post`, `Delete`, `Put`, `Patch`, `Options`, `Connect`, `Trace`, and `Head` are available out of the box.

```php
use Tempest\Router\Get;
use Tempest\View\View;
use function Tempest\View\view;

final readonly class HomeController
{
    #[Get(uri: '/home')]
    public function __invoke(): View
    {
        return view('./home.view.php');
    }
}
```

## Route Parameters

Dynamic segments wrapped in curly braces become method parameters:

```php
#[Get(uri: '/aircraft/{id}')]
public function show(int $id): View
{
    $aircraft = $this->aircraftRepository->getAircraftById($id);
    return view('./aircraft.view.php', aircraft: $aircraft);
}
```

### Optional Parameters

Parameters can be marked optional with a `?` prefix. The corresponding method parameter must be nullable or have a default value:

```php
#[Get(uri: '/aircraft/{?id}')]
public function index(?string $id): View
{
    if ($id === null) {
        $aircraft = $this->aircraftRepository->all();
    } else {
        $aircraft = $this->aircraftRepository->find($id);
    }
    return view('aircraft.view.php', aircraft: $aircraft);
}
```

Multiple optional parameters work with defaults or nullability:

```php
#[Get(uri: '/aircraft/{?manufacturer}/{?model}')]
public function search(?string $manufacturer, ?string $model): View
{
    // Matches /aircraft, /aircraft/pilatus, and /aircraft/pilatus/pc24
}
```

Optional parameters support regular expression constraints:

```php
#[Get(uri: '/aircraft/{?id:\d+}')]
public function show(?int $id): View
{
    // Matches /aircraft and /aircraft/123
}
```

### Regular Expression Constraints

Constrain parameter formats using regex patterns after the parameter name:

```php
#[Get(uri: '/aircraft/{id:[0-9]+}')]
public function showAircraft(int $id): View
{
    // …
}
```

## Route Binding

Controller actions can receive objects instead of scalar identifiers. Classes implementing the `Bindable` interface provide a static `resolve()` method:

```php
use Tempest\Router\Bindable;

final class Aircraft implements Bindable
{
    public static function resolve(string $input): self
    {
        return query(self::class)->resolve($input);
    }
}
```

Use the `IsBindingValue` attribute to customize which property is used for URI generation:

```php
use Tempest\Router\Bindable;
use Tempest\Router\IsBindingValue;

final class Aircraft implements Bindable
{
    #[IsBindingValue]
    public string $registrationNumber;

    public static function resolve(string $input): self
    {
        return query(self::class)
            ->where('registrationNumber', $input)
            ->first();
    }
}
```

## Backed Enum Binding

String-backed enumerations can be injected directly. Tempest maps URI parameters to enum instances using `tryFrom`:

```php
enum AircraftType: string
{
    case PC12 = 'pc12';
    case PC24 = 'pc24';
    case SF50 = 'sf50';
}

final readonly class AircraftController
{
    #[Get('/aircraft/{type}')]
    public function show(AircraftType $type): Response { /* … */ }
}
```

If the parameter doesn't match an enum case, a 404 response is returned.

## Generating URIs

The `uri()` function generates URIs to controller methods:

```php
use function Tempest\Router\uri;

// Invokable classes
uri(HomeController::class);
// → /home

// Classes with named methods
uri([AircraftController::class, 'store']);
// → /aircraft

// With parameters
uri([AircraftController::class, 'show'], id: $aircraft->id);
// → /aircraft/1
```

### Signed URIs

Create tamper-proof URIs using `signed_uri()`:

```php
use function Tempest\Router\signed_uri;

signed_uri(
    action: [MailingListController::class, 'unsubscribe'],
    email: $email
);
```

For time-limited URIs:

```php
use function Tempest\Router\temporary_signed_uri;

temporary_signed_uri(
    action: PasswordlessAuthenticationController::class,
    duration: Duration::minutes(10),
    userId: $userId
);
```

Verify signatures with the `UriGenerator` class:

```php
final class PasswordlessAuthenticationController
{
    public function __construct(
        private readonly UriGenerator $uri,
    ) {}

    public function __invoke(Request $request): Response
    {
        if (! $this->uri->hasValidSignature($request)) {
            throw new HttpRequestFailed(Status::UNPROCESSABLE_CONTENT);
        }
        // …
    }
}
```

### Matching the Current URI

Check if the current request matches a controller action:

```php
use function Tempest\Router\is_current_uri;

// GET /aircraft/1
is_current_uri(AircraftController::class); // true
is_current_uri(AircraftController::class, id: 1); // true
is_current_uri(AircraftController::class, id: 2); // false
```

## Accessing Request Data

### Using Request Classes

Structured data is handled through request classes implementing `Request` with the `IsRequest` trait:

```php
use Tempest\Http\Request;
use Tempest\Http\IsRequest;
use Tempest\Validation\Rules\HasLength;

final class RegisterAirportRequest implements Request
{
    use IsRequest;

    #[HasLength(min: 10, max: 120)]
    public string $name;

    #[HasLength(min: 2)]
    public string $servedCity;

    #[HasLength(min: 4, max: 4)]
    public string $icaoCode;

    public ?DateTime $registeredAt = null;
}
```

Inject into controller actions:

```php
final readonly class AirportController
{
    #[Post(uri: '/airports/register')]
    public function store(RegisterAirportRequest $request): Redirect
    {
        $airport = map($request)
            ->to(Airport::class)
            ->save();

        return new Redirect(uri([self::class, 'show'], id: $airport->id));
    }
}
```

### Sensitive Fields

Mark properties as sensitive to prevent re-population in forms after validation errors:

```php
use Tempest\Http\SensitiveField;

final class ResetPasswordRequest implements Request
{
    use IsRequest;

    public string $email;

    #[SensitiveField]
    #[HasLength(min: 8)]
    public string $password;
}
```

### Retrieving Data Directly

For simpler cases, use the `Request` object's `get` method:

```php
final readonly class AircraftController
{
    #[Get(uri: '/aircraft')]
    public function me(Request $request): View
    {
        $icao = $request->get('icao');
        // …
    }
}
```

## Form Validation

Tempest automatically validates request objects using type hints and validation attributes. On failure, responses either redirect (web) or return 422 (stateless). Errors are available:

- In the `X-Validation` header as JSON
- Through the `FormSession` class

Built-in view components display errors:

```html
<x-form :action="uri(StorePostController::class)">
  <x-input name="name" />
  <x-input type="email" name="email" />
  <x-submit />
</x-form>
```

Install custom versions: `./tempest install view-components`

## Route Middleware

Apply middleware to routes via the `middleware` parameter:

```php
final readonly class ReceiveInteractionController
{
    #[Post('/slack/interaction', middleware: [ValidateWebhook::class])]
    public function __invoke(): Response
    {
        // …
    }
}
```

Middleware must implement `HttpMiddleware`:

```php
use Tempest\Router\HttpMiddleware;
use Tempest\Router\HttpMiddlewareCallable;
use Tempest\Http\Request;
use Tempest\Http\Response;
use Tempest\Discovery\SkipDiscovery;
use Tempest\Core\Priority;

#[SkipDiscovery]
#[Priority(Priority::LOW)]
final readonly class ValidateWebhook implements HttpMiddleware
{
    public function __invoke(Request $request, HttpMiddlewareCallable $next): Response
    {
        $signature = $request->headers->get('X-Slack-Signature');
        $timestamp = $request->headers->get('X-Slack-Request-Timestamp');
        // …
        return $next($request);
    }
}
```

### Middleware Priority

Control execution order with the `Priority` attribute:

```php
use Tempest\Core\Priority;

#[Priority(Priority::HIGH)]
final readonly class ValidateWebhook implements HttpMiddleware
{ /* … */ }
```

Use constants: `Priority::HIGH`, `Priority::NORMAL` (default), `Priority::LOW`.

### Middleware Discovery

Global middleware is automatically discovered and sorted. Skip with `#[SkipDiscovery]`:

```php
use Tempest\Discovery\SkipDiscovery;

#[SkipDiscovery]
final readonly class ValidateWebhook implements HttpMiddleware
{ /* … */ }
```

### Cross-Site Request Forgery Protection

The `PreventCrossSiteRequestsMiddleware` (included by default) uses browser-generated `Sec-Fetch-*` headers that cannot be forged:

- `Sec-Fetch-Site`: indicates request origin (same domain, different site, etc.)
- `Sec-Fetch-Mode`: distinguishes navigation from resource loading

This works with modern browsers. Exclude and implement traditional token-based CSRF protection for older browser support.

### Excluding Route Middleware

Skip specific middleware for certain routes:

```php
final readonly class HealthCheckController
{
    #[Get('/health', without: [RateLimitMiddleware::class])]
    public function __invoke(): Response
    {
        return new Ok(['status' => 'healthy']);
    }
}
```

## Route Decorators

Decorators apply common configuration to controllers or methods. Built-in options include:

### `#[Stateless]`

Remove session and cookie overhead for APIs and feeds:

```php
final readonly class BlogPostController
{
    #[Stateless]
    #[Get('/rss')]
    public function rss(): Response { /* … */ }
}
```

### `#[Prefix]`

Add a URI prefix to all routes:

```php
#[Prefix('/api')]
final readonly class ApiController
{
    #[Get('/books')]
    public function books(): Response { /* … */ }

    #[Get('/authors')]
    public function authors(): Response { /* … */ }
}
```

### `#[WithMiddleware]`

Add middleware to all routes:

```php
#[WithMiddleware(AuthMiddleware::class, AdminMiddleware::class)]
final readonly class AdminController { /* … */ }
```

### `#[WithoutMiddleware]`

Remove middleware from all routes:

```php
#[WithoutMiddleware(PreventCrossSiteRequestsMiddleware::class)]
final readonly class StatelessController { /* … */ }
```

### Custom Route Decorators

Implement `RouteDecorator`:

```php
use Attribute;
use Tempest\Router\RouteDecorator;

#[Attribute(Attribute::TARGET_METHOD | Attribute::TARGET_CLASS)]
final readonly class Auth implements RouteDecorator
{
    public function decorate(Route $route): Route
    {
        $route->middleware[] = AuthMiddleware::class;
        return $route;
    }
}
```

## Responses

Controllers return `View` or `Response` objects. Scalar values and arrays convert automatically.

### View Responses

Return views directly:

```php
final readonly class AircraftController
{
    #[Get(uri: '/aircraft/{aircraft}')]
    public function show(Aircraft $aircraft, User $user): View
    {
        return view('./show.view.php',
            aircraft: $aircraft,
            user: $user,
        );
    }
}
```

### Built-in Response Classes

- `Ok` — 200 response with optional body
- `Created` — 201 response with optional body
- `Redirect` — redirect to specified URI
- `Back` — redirect to previous page with fallback
- `Download` — download file to browser
- `File` — display file in browser
- `NotFound` — 404 response with optional body
- `ServerError` — 500 response

Example:

```php
final readonly class FlightPlanController
{
    #[Get('/{flight}/flight-plan/download')]
    public function download(Flight $flight): Response
    {
        if (! $this->accessControl->isGranted('view', $flight)) {
            return new Redirect('/');
        }
        return new Download($flight->flight_plan_path);
    }
}
```

### Sending Generic Responses

Use `GenericResponse` for dynamic status codes:

```php
final readonly class CreateFlightController
{
    #[Post('/{flight}')]
    public function __invoke(Flight $flight): Response
    {
        $status = /* … */
        $body = /* … */

        return new GenericResponse(
            status: $status,
            body: $body,
        );
    }
}
```

### Custom Response Classes

Implement `Response` using the `IsResponse` trait:

```php
use Tempest\Http\IsResponse;
use Tempest\Http\Response;
use Tempest\Http\Status;

final class AircraftRegistered implements Response
{
    use IsResponse;

    public function __construct(Aircraft $aircraft)
    {
        $this->status = Status::CREATED;
        $this->flash(
            key: 'success',
            value: "Aircraft {$aircraft->icao_code} was successfully registered."
        );
    }
}
```

### Specifying Content Types

Override inferred content types:

```php
final readonly class JsonController
{
    #[Get('/json')]
    public function json(string $path): Response
    {
        $data = [ /* … */ ];
        return new Ok($data)->setContentType(ContentType::JSON);
    }
}
```

### Post-Processing Responses

Implement `ResponseProcessor` for response transformations:

```php
use function Tempest\View\view;

final readonly class ErrorResponseProcessor implements ResponseProcessor
{
    public function process(Response $response): Response
    {
        if (! $response->status->isSuccessful()) {
            return $response->setBody(view('./error.view.php', status: $response->status));
        }
        return $response;
    }
}
```

## Session Management

Inject the `Session` class (automatically started):

```php
use Tempest\Http\Session\Session;

final readonly class TodoController
{
    public function __construct(
        private Session $session,
    ) {}

    #[Post('/select/{todo}')]
    public function select(Todo $todo): View
    {
        if ($this->session->get('selected_todo') === $todo->id) {
            $this->session->remove('selected_todo');
        } else {
            $this->session->set('selected_todo', $todo->id);
        }
        return $this->list();
    }
}
```

### Flashing Values

Store values for the next request only:

```php
public function store(Todo $todo): Redirect
{
    $this->session->flash('message', value: 'Save was successful');
    return new Redirect('/');
}
```

### Session Configuration

Create `session.config.php` to configure driver and expiration.

#### File Sessions (default)

```php
use Tempest\Http\Session\Config\FileSessionConfig;
use Tempest\DateTime\Duration;

return new FileSessionConfig(
   expiration: Duration::days(30),
   path: 'sessions',
);
```

#### Database Sessions

Install: `./tempest install sessions:database`

```php
use Tempest\Http\Session\Config\DatabaseSessionConfig;
use Tempest\DateTime\Duration;

return new DatabaseSessionConfig(
    expiration: Duration::days(30),
);
```

### Session Cleaning

Sessions expire based on last activity. Clean outdated sessions with:

```bash
./tempest session:clean
```

Runs automatically with scheduling enabled.

## Deferring Tasks

Execute tasks after response completion:

```php
use function Tempest\defer;
use function Tempest\event;

final readonly class TrackVisitMiddleware implements HttpMiddleware
{
    public function __invoke(Request $request, HttpMiddlewareCallable $next): Response
    {
        defer(fn () => event(new PageVisited($request->getUri())));
        return $next($request);
    }
}
```

Deferred callbacks receive injected dependencies.

**Note:** Requires `fastcgi_finish_request()` for true after-response execution.

## Testing

Router testing uses the `http` property of `IntegrationTest`:

```php
final class ProfileControllerTest extends IntegrationTestCase
{
    public function test_can_render_profile(): void
    {
        $response = $this->http
            ->get('/account/profile')
            ->assertOk()
            ->assertSee('My Profile');
    }
}
```

The `TestResponseHelper` provides assertion methods for all HTTP verbs.
