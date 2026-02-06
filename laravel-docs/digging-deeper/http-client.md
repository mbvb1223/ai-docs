# Laravel 12.x HTTP Client Documentation

## Introduction

Laravel provides an expressive, minimal API around the Guzzle HTTP client for making outgoing HTTP requests.

## Making Requests

### Basic Requests

```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
$response = Http::post('http://example.com/users', ['name' => 'Steve']);
$response = Http::put('http://example.com/users/1', ['name' => 'Steve']);
$response = Http::patch('http://example.com/users/1', ['name' => 'Steve']);
$response = Http::delete('http://example.com/users/1');
```

### Response Methods

```php
$response->body();          // string
$response->json();          // mixed
$response->object();        // object
$response->collect();       // Collection
$response->status();        // int
$response->successful();    // bool (2xx)
$response->failed();        // bool (4xx or 5xx)
$response->clientError();   // bool (4xx)
$response->serverError();   // bool (5xx)
$response->header($header); // string
$response->headers();       // array
```

### Status Code Helpers

```php
$response->ok();                  // 200
$response->created();             // 201
$response->accepted();            // 202
$response->noContent();           // 204
$response->badRequest();          // 400
$response->unauthorized();        // 401
$response->forbidden();           // 403
$response->notFound();            // 404
$response->conflict();            // 409
$response->unprocessableEntity(); // 422
$response->tooManyRequests();     // 429
$response->serverError();         // 500
```

### URI Templates

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '12.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

## Request Data

### GET Query Parameters

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);

// Or using withQueryParameters
Http::withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users');
```

### Form URL Encoded

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
]);
```

### Raw Request Body

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

### File Uploads

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```

## Headers

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);

// Content type
$response = Http::accept('application/json')->get('http://example.com/users');
$response = Http::acceptJson()->get('http://example.com/users');
```

## Authentication

```php
// Basic authentication
$response = Http::withBasicAuth('[email protected]', 'secret')->post(/* ... */);

// Digest authentication
$response = Http::withDigestAuth('[email protected]', 'secret')->post(/* ... */);

// Bearer token
$response = Http::withToken('token')->post(/* ... */);
```

## Timeout

```php
// Request timeout (default: 30 seconds)
$response = Http::timeout(3)->get(/* ... */);

// Connection timeout (default: 10 seconds)
$response = Http::connectTimeout(3)->get(/* ... */);
```

## Retries

```php
// Basic retry
$response = Http::retry(3, 100)->post(/* ... */);

// Custom retry delay
$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);

// Array of delays
$response = Http::retry([100, 200])->post(/* ... */);

// Conditional retries
$response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```

## Error Handling

### Throwing Exceptions

```php
$response = Http::post(/* ... */);

$response->throw();
$response->throwIf($condition);
$response->throwUnless($condition);
$response->throwIfStatus(403);
$response->throwUnlessStatus(200);

// Chain after throw
return Http::post(/* ... */)->throw()->json();
```

## Guzzle Middleware

### Request Middleware

```php
$response = Http::withRequestMiddleware(
    function (RequestInterface $request) {
        return $request->withHeader('X-Example', 'Value');
    }
)->get('http://example.com');
```

### Response Middleware

```php
$response = Http::withResponseMiddleware(
    function (ResponseInterface $response) {
        return $response->withHeader('X-Finished-At', now()->toDateTimeString());
    }
)->get('http://example.com');
```

### Global Middleware

```php
Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

## Concurrent Requests

### Request Pooling

```php
use Illuminate\Http\Client\Pool;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() && $responses[1]->ok() && $responses[2]->ok();
```

### Named Requests

```php
$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->get('http://localhost/first'),
    $pool->as('second')->get('http://localhost/second'),
]);

return $responses['first']->ok();
```

### Request Batching

```php
$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
])->then(function (Batch $batch, array $results) {
    // All requests completed successfully...
})->catch(function (Batch $batch, int|string $key, $response) {
    // Batch request failure detected...
})->finally(function (Batch $batch, array $results) {
    // The batch has finished executing...
})->send();
```

## Macros

```php
Http::macro('github', function () {
    return Http::withHeaders([
        'X-Example' => 'example',
    ])->baseUrl('https://github.com');
});

// Usage
$response = Http::github()->get('/');
```

## Testing

### Faking Requests

```php
use Illuminate\Support\Facades\Http;

Http::fake();

$response = Http::post(/* ... */);
```

### Faking Specific URLs

```php
Http::fake([
    'github.com/*' => Http::response(['foo' => 'bar'], 200),
    'google.com/*' => Http::response('Hello World', 200),
    '*' => Http::response('Hello World', 200),
]);
```

### Response Sequences

```php
Http::fake([
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->pushStatus(404),
]);
```

### Inspecting Requests

```php
Http::fake();

Http::withHeaders(['X-First' => 'foo'])
    ->post('http://example.com/users', ['name' => 'Taylor']);

Http::assertSent(function (Request $request) {
    return $request->hasHeader('X-First', 'foo') &&
           $request->url() == 'http://example.com/users' &&
           $request['name'] == 'Taylor';
});

Http::assertNotSent(function (Request $request) {
    return $request->url() === 'http://example.com/posts';
});

Http::assertSentCount(5);
Http::assertNothingSent();
```

### Preventing Stray Requests

```php
Http::preventStrayRequests();

Http::fake([
    'github.com/*' => Http::response('ok'),
]);

Http::get('https://github.com/laravel/framework'); // OK
Http::get('https://laravel.com'); // Exception thrown
```

## Events

- `RequestSending` - before request is sent
- `ResponseReceived` - after response is received
- `ConnectionFailed` - when no response is received

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/http-client)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
