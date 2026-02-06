# Laravel 12.x HTTP Responses Documentation

## Introduction

Laravel provides various ways to create HTTP responses, from simple strings to complex JSON and streamed responses.

## Creating Responses

### Strings and Arrays

```php
Route::get('/', function () {
    return 'Hello World';
});

Route::get('/', function () {
    return [1, 2, 3];  // Auto-converted to JSON
});
```

### Response Objects

```php
return response('Hello World', 200)
    ->header('Content-Type', 'text/plain');
```

### Eloquent Models

```php
Route::get('/user/{user}', function (User $user) {
    return $user;  // Auto-converted to JSON
});
```

## Attaching Headers

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Value');

return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Value',
    ]);
```

## Attaching Cookies

```php
return response('Hello World')
    ->cookie('name', 'value', $minutes);

// Expire cookie
return response('Hello World')->withoutCookie('name');
```

## Redirects

```php
return redirect('/home/dashboard');
return back()->withInput();

// Named routes
return redirect()->route('login');
return redirect()->route('profile', ['id' => 1]);
return redirect()->route('profile', [$user]);

// Controller actions
return redirect()->action([UserController::class, 'index']);

// External domains
return redirect()->away('https://www.google.com');

// With flashed data
return redirect('/dashboard')->with('status', 'Profile updated!');
```

## View Responses

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

## JSON Responses

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);

// JSONP
return response()
    ->json(['name' => 'Abigail'])
    ->withCallback($request->input('callback'));
```

## File Downloads

```php
return response()->download($pathToFile);
return response()->download($pathToFile, $name, $headers);
```

## File Responses

```php
return response()->file($pathToFile);
```

## Streamed Responses

```php
return response()->stream(function (): void {
    foreach (['developer', 'admin'] as $string) {
        echo $string;
        ob_flush();
        flush();
        sleep(2);
    }
}, 200, ['X-Accel-Buffering' => 'no']);
```

### Using Generators

```php
return response()->stream(function (): Generator {
    $stream = OpenAI::client()->chat()->createStreamed(...);
    foreach ($stream as $response) {
        yield $response->choices[0];
    }
});
```

### Streamed JSON

```php
return response()->streamJson([
    'users' => User::cursor(),
]);
```

### Event Streams (SSE)

```php
return response()->eventStream(function () {
    foreach ($stream as $response) {
        yield $response->choices[0];
    }
});
```

### Frontend Consumption (React/Vue)

```bash
npm install @laravel/stream-react
npm install @laravel/stream-vue
```

```jsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");
    return <div>{data}</div>;
}
```

## Response Macros

```php
use Illuminate\Support\Facades\Response;

Response::macro('caps', function (string $value) {
    return Response::make(strtoupper($value));
});

// Usage
return response()->caps('foo');
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/responses)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
