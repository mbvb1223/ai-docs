# Laravel 12.x Passport Documentation

## Introduction

Laravel Passport is a full OAuth2 server implementation for Laravel applications, built on top of the League OAuth2 server.

**Passport or Sanctum?**
- Use **Passport** if your application needs OAuth2 support
- Use **Sanctum** for SPAs, mobile apps, or simple API token issuance

## Installation

```bash
php artisan install:api --passport
```

Add trait to User model:

```php
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens;
}
```

Configure API guard in `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

### Deploying

```bash
php artisan passport:keys
```

Or via environment variables:

```env
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----..."
PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----..."
```

## Configuration

### Token Lifetimes

```php
use Carbon\CarbonInterval;

public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

## Authorization Code Grant

### Creating Clients

```bash
php artisan passport:client
```

### Requesting Tokens

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

### Converting Codes to Tokens

```php
Route::get('/callback', function (Request $request) {
    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

### Refreshing Tokens

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
]);
```

## PKCE Grant

For SPAs and mobile apps without client secrets.

```bash
php artisan passport:client --public
```

```php
$codeChallenge = strtr(rtrim(
    base64_encode(hash('sha256', $codeVerifier, true))
, '='), '+/', '-_');

$query = http_build_query([
    'client_id' => 'your-client-id',
    'redirect_uri' => 'https://app.com/callback',
    'response_type' => 'code',
    'scope' => 'user:read',
    'code_challenge' => $codeChallenge,
    'code_challenge_method' => 'S256',
]);
```

## Device Authorization Grant

For TVs, game consoles, and browserless devices.

```bash
php artisan passport:client --device
```

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read',
]);
// Returns: device_code, user_code, verification_uri
```

## Client Credentials Grant

For machine-to-machine authentication.

```bash
php artisan passport:client --client
```

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read',
]);
```

## Personal Access Tokens

```bash
php artisan passport:client --personal
```

```php
$token = $user->createToken('My Token')->accessToken;
$token = $user->createToken('My Token', ['user:read'])->accessToken;
```

## Token Scopes

### Defining Scopes

```php
Passport::tokensCan([
    'user:read' => 'Retrieve user info',
    'orders:create' => 'Place orders',
]);

Passport::defaultScopes(['user:read']);
```

### Checking Scopes

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // ...
})->middleware(['auth:api', CheckToken::using('orders:read')]);
```

```php
if ($request->user()->tokenCan('orders:create')) {
    // ...
}
```

## Protecting Routes

```php
Route::get('/user', function () {
    // ...
})->middleware('auth:api');
```

### Passing Access Token

```php
$response = Http::withHeaders([
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');
```

## SPA Authentication

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

## Testing

```php
use Laravel\Passport\Passport;

test('orders can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');
    $response->assertStatus(201);
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/passport)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
