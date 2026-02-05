# OAuth

## Overview

Tempest enables user authentication through multiple OAuth providers including GitHub, Google, Discord, Facebook, Instagram, LinkedIn, Microsoft, Slack, and Apple. The implementation leverages the PHP League's OAuth 2.0 client library for reliability and battle-tested functionality.

## Getting Started

### Quick Setup

Run the installer command to quickly configure OAuth:

```bash
./tempest install auth --oauth
```

The installer will:
- Prompt selection of OAuth providers
- Publish configuration files and controller stubs
- Add OAuth credentials to `.env` and `.env.example`
- Install required Composer dependencies

### Manual Configuration

Create a provider configuration file (e.g., `app/Auth/github.config.php`):

```php
return new GitHubOAuthConfig(
    clientId: env('GITHUB_CLIENT_ID'),
    clientSecret: env('GITHUB_CLIENT_SECRET'),
    redirectTo: [GitHubOAuthController::class, 'callback'],
    scopes: ['user:email'],
);
```

Interact with OAuth using the `OAuthClient` interface through dependency injection.

## Implementing the OAuth Flow

A complete OAuth flow requires two routes:

1. **Authorization route** - redirects users to the provider's authorization page
2. **Callback route** - handles the provider's response and user authentication

### Example Controller Implementation

```php
use Tempest\Auth\OAuth\OAuthClient;

final readonly class DiscordOAuthController
{
    public function __construct(
        private OAuthClient $oauth,
        private Session $session,
        private Authenticator $authenticator,
    ) {}

    #[Get('/auth/discord')]
    public function redirect(): Redirect
    {
        return $this->oauth->createRedirect(scopes: ['identify']);
    }

    #[Get('/auth/discord/callback')]
    public function callback(Request $request): Redirect
    {
        $user = $this->oauth->authenticate(
            request: $request,
            map: fn (OAuthUser $user): User => query(User::class)->updateOrCreate([
                'discord_id' => $user->id,
            ], [
                'discord_id' => $user->id,
                'username' => $user->nickname,
                'email' => $user->email,
            ])
        );

        return new Redirect('/');
    }
}
```

## Working with the OAuth User

The `OAuthUser` object contains user information from the OAuth provider:

```php
$user = $this->oauth->fetchUser($code);

$user->id;         // Unique identifier from provider
$user->email;      // User's email address
$user->name;       // User's full name
$user->nickname;   // Username/nickname
$user->avatar;     // Avatar URL
$user->provider;   // OAuth provider name
$user->raw;        // Raw provider data
```

## Supporting Multiple Providers

Use tags to work with multiple OAuth providers simultaneously:

```php
enum Provider
{
    case GITHUB;
    case GOOGLE;
    case DISCORD;
}
```

Configure each provider with a unique tag:

```php
// github.config.php
return new GitHubOAuthConfig(
    tag: Provider::GITHUB,
    clientId: env('GITHUB_CLIENT_ID'),
    clientSecret: env('GITHUB_CLIENT_SECRET'),
    redirectTo: [OAuthController::class, 'handleGitHubCallback'],
    scopes: ['user:email'],
);
```

Inject providers using the `Tag` attribute:

```php
final readonly class AuthController
{
    public function __construct(
        #[Tag(Provider::GITHUB)]
        private OAuthClient $githubClient,
        #[Tag(Provider::GOOGLE)]
        private OAuthClient $googleClient,
    ) {}
}
```

## Using a Generic Provider

For unsupported OAuth providers, use `GenericOAuthConfig`:

```php
return new GenericOAuthConfig(
    clientId: env('CUSTOM_CLIENT_ID'),
    clientSecret: env('CUSTOM_CLIENT_SECRET'),
    redirectTo: [OAuthController::class, 'handleCallback'],
    urlAuthorize: 'https://provider.com/oauth/authorize',
    urlAccessToken: 'https://provider.com/oauth/token',
    urlResourceOwnerDetails: 'https://provider.com/api/user',
    scopes: ['read:user'],
);
```

## Available Providers

Tempest includes configuration classes for:

- **GitHub** - `GitHubOAuthConfig`
- **Google** - `GoogleOAuthConfig`
- **Facebook** - `FacebookOAuthConfig`
- **Discord** - `DiscordOAuthConfig`
- **Instagram** - `InstagramOAuthConfig`
- **LinkedIn** - `LinkedInOAuthConfig`
- **Microsoft** - `MicrosoftOAuthConfig`
- **Slack** - `SlackOAuthConfig`
- **Apple** - `AppleOAuthConfig`
- **Generic** - `GenericOAuthConfig` (any platform)

## Testing

### Faking an OAuth Client

Extend `IntegrationTest` to access OAuth testing utilities:

```php
final class OAuthControllerTest extends IntegrationTestCase
{
    #[Test]
    public function oauth(): void
    {
        $oauth = $this->oauth->fake(new OAuthUser(
            id: 'jon',
            email: 'jondoe@example.test',
            nickname: 'jondoe',
        ));

        $this->http
            ->get('/oauth/discord')
            ->assertRedirect($oauth->lastAuthorizationUrl);

        $oauth->assertAuthorizationUrlGenerated();

        $this->http
            ->get("/oauth/discord/callback", query: [
                'code' => 'some-fake-code',
                'state' => $oauth->getState()
            ])
            ->assertRedirect('/');

        $oauth->assertUserFetched(code: 'some-fake-code');
    }
}
```
