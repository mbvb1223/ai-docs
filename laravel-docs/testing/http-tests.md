# Laravel 12.x HTTP Testing Documentation

## Introduction

Laravel provides a fluent API for making HTTP requests and examining responses.

```php
test('the application returns a successful response', function () {
    $response = $this->get('/');
    $response->assertStatus(200);
});
```

## Making Requests

Available methods: `get()`, `post()`, `put()`, `patch()`, `delete()`

### Customizing Request Headers

```php
$response = $this->withHeaders([
    'X-Header' => 'Value',
])->post('/user', ['name' => 'Sally']);
```

### Cookies

```php
$response = $this->withCookie('color', 'blue')->get('/');

$response = $this->withCookies([
    'color' => 'blue',
    'name' => 'Taylor',
])->get('/');
```

### Session / Authentication

```php
$response = $this->withSession(['banned' => false])->get('/');

$user = User::factory()->create();
$response = $this->actingAs($user)->get('/');

$this->actingAs($user, 'web');
$this->actingAsGuest();
```

### Debugging Responses

```php
$response->dump();
$response->dumpHeaders();
$response->dumpSession();

$response->dd();
$response->ddHeaders();
$response->ddBody();
$response->ddJson();
```

### Exception Handling

```php
use Illuminate\Support\Facades\Exceptions;

Exceptions::fake();
$response = $this->get('/order/1');
Exceptions::assertReported(InvalidOrderException::class);

$response = $this->withoutExceptionHandling()->get('/');
```

## Testing JSON APIs

```php
$response = $this->postJson('/api/user', ['name' => 'Sally']);

$response
    ->assertStatus(201)
    ->assertJson([
        'created' => true,
    ]);
```

### Exact JSON Matches

```php
$response->assertExactJson([
    'created' => true,
]);
```

### JSON Paths

```php
$response->assertJsonPath('team.owner.name', 'Darian');
```

### Fluent JSON Testing

```php
use Illuminate\Testing\Fluent\AssertableJson;

$response->assertJson(fn (AssertableJson $json) =>
    $json->where('id', 1)
        ->where('name', 'Victoria Faith')
        ->whereNot('status', 'pending')
        ->missing('password')
        ->etc()
);
```

### JSON Collections

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has(3)
        ->first(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->etc()
        )
);
```

## Testing File Uploads

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

Storage::fake('avatars');

$file = UploadedFile::fake()->image('avatar.jpg');

$response = $this->post('/avatar', [
    'avatar' => $file,
]);

Storage::disk('avatars')->assertExists($file->hashName());
Storage::disk('avatars')->assertMissing('missing.jpg');
```

### Fake File Customization

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes, 'application/pdf');
```

## Testing Views

```php
$view = $this->view('welcome', ['name' => 'Taylor']);
$view->assertSee('Taylor');
```

### Rendering Blade

```php
$view = $this->blade('<x-component :name="$name" />', ['name' => 'Taylor']);
$view = $this->component(Profile::class, ['name' => 'Taylor']);
```

## Available Assertions

### Response Status

- `assertOk()` - 200
- `assertCreated()` - 201
- `assertAccepted()` - 202
- `assertNoContent()` - 204
- `assertFound()` - 302
- `assertBadRequest()` - 400
- `assertUnauthorized()` - 401
- `assertForbidden()` - 403
- `assertNotFound()` - 404
- `assertUnprocessable()` - 422
- `assertTooManyRequests()` - 429
- `assertInternalServerError()` - 500
- `assertStatus($code)`

### Headers & Cookies

```php
$response->assertHeader($headerName, $value = null);
$response->assertCookie($cookieName, $value = null);
$response->assertCookieMissing($cookieName);
```

### Content

```php
$response->assertSee($value);
$response->assertSeeText($value);
$response->assertDontSee($value);
$response->assertContent($value);
$response->assertDownload($filename = null);
```

### JSON

```php
$response->assertJson(array $data);
$response->assertExactJson(array $data);
$response->assertJsonCount($count, $key = null);
$response->assertJsonFragment(array $data);
$response->assertJsonPath($path, $expectedValue);
$response->assertJsonStructure(array $structure);
$response->assertJsonValidationErrors(array $data);
```

### Redirects

```php
$response->assertRedirect($uri = null);
$response->assertRedirectToRoute($name, $parameters = []);
$response->assertRedirectContains($string);
```

### Session

```php
$response->assertSessionHas($key, $value = null);
$response->assertSessionHasErrors(array $keys = []);
$response->assertSessionHasNoErrors();
$response->assertSessionMissing($key);
```

### View

```php
$response->assertViewHas($key, $value = null);
$response->assertViewIs($value);
$response->assertViewMissing($key);
```

### Validation

```php
$response->assertValid(['name', 'email']);
$response->assertInvalid(['name', 'email']);
```

### Authentication

```php
$this->assertAuthenticated($guard = null);
$this->assertGuest($guard = null);
$this->assertAuthenticatedAs($user, $guard = null);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/http-tests)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
