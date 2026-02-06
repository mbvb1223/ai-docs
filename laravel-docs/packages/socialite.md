# Laravel 12.x Socialite OAuth Documentation

## Introduction

Laravel Socialite provides simple OAuth authentication with Facebook, X (Twitter), LinkedIn, Google, GitHub, GitLab, Bitbucket, and Slack.

## Installation

```bash
composer require laravel/socialite
```

## Configuration

Add credentials to `config/services.php`:

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

## Authentication

### Routing

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();
});
```

### Authentication and Storage

```php
Route::get('/auth/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

### Access Scopes

```php
return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();

// Overwrite all scopes
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

### Optional Parameters

```php
return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

## Retrieving User Details

```php
$user = Socialite::driver('github')->user();

// OAuth 2.0 providers
$token = $user->token;
$refreshToken = $user->refreshToken;
$expiresIn = $user->expiresIn;

// All providers
$user->getId();
$user->getNickname();
$user->getName();
$user->getEmail();
$user->getAvatar();
```

### From Token

```php
$user = Socialite::driver('github')->userFromToken($token);
```

### Stateless Authentication

```php
return Socialite::driver('google')->stateless()->user();
```

## Testing

### Faking Redirect

```php
Socialite::fake('github');

$response = $this->get('/auth/github/redirect');
$response->assertRedirect();
```

### Faking Callback

```php
use Laravel\Socialite\Two\User;

Socialite::fake('github', (new User)->map([
    'id' => 'github-123',
    'name' => 'Jason Beggs',
    'email' => '[email protected]',
]));

$response = $this->get('/auth/github/callback');
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/socialite)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
