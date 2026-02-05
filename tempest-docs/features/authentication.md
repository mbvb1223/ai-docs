# Authentication

## Overview

Tempest provides a flexible authentication system that doesn't assume authenticatable models must be users. This allows implementation for API keys, service accounts, or other authentication systems. The framework also includes policy-based access control for fine-grained permissions.

## Quick Start

To establish basic user authentication:

```sh
./tempest install auth
./tempest migrate
```

## Authentication Implementation

### Creating an Authenticatable Model

Implement the `Authenticatable` interface for any model requiring authentication:

```php
use Tempest\Auth\Authentication\Authenticatable;
use Tempest\Database\PrimaryKey;
use Tempest\Database\Hashed;

final class User implements Authenticatable
{
    public PrimaryKey $id;

    public function __construct(
        public string $email,
        #[Hashed]
        #[\SensitiveParameter]
        public ?string $password,
    ) {}
}
```

### Authenticating Models

Use the `Authenticator` service in controllers to handle authentication logic:

```php
use Tempest\Auth\Authentication\Authenticator;
use Tempest\Cryptography\Password\PasswordHasher;

final readonly class AuthenticationController
{
    public function __construct(
        private Authenticator $authenticator,
        private PasswordHasher $passwordHasher,
    ) {}

    #[Post('/login')]
    public function login(LoginRequest $request): Redirect
    {
        $user = query(User::class)
            ->select()
            ->where('email', $request->email)
            ->first();

        if (! $user || ! $this->passwordHasher->verify($request->password, $user->password)) {
            return new Redirect('/login')->flash('error', 'Invalid credentials');
        }

        $this->authenticator->authenticate($user);
        return new Redirect('/');
    }

    #[Post('/logout')]
    public function logout(): Redirect
    {
        $this->authenticator->deauthenticate();
        return new Redirect('/login');
    }
}
```

### Accessing Authenticated Models

Inject the `Authenticator` to retrieve the current authenticated model:

```php
final readonly class ProfileController
{
    public function __construct(
        private Authenticator $authenticator,
    ) {}

    #[Get('/profile', middleware: [MustBeAuthenticated::class])]
    public function show(): View
    {
        return view('profile.view.php', user: $this->authenticator->current());
    }
}
```

Alternatively, inject the authenticatable model directly:

```php
final readonly class ProfileController
{
    public function __construct(
        private User $user,
    ) {}

    #[Get('/profile', middleware: [MustBeAuthenticated::class])]
    public function show(): View
    {
        return view('profile.view.php', user: $this->user);
    }
}
```

## Custom Authenticatable Resolvers

Implement the `AuthenticatableResolver` interface to fetch models from custom sources like LDAP or external APIs:

```php
use Tempest\Auth\Authentication\AuthenticatableResolver;

final readonly class LdapAuthenticatableResolver implements AuthenticatableResolver
{
    public function __construct(
        private LdapClient $ldap,
    ) {}

    public function resolve(int|string $id, string $class): ?Authenticatable
    {
        $attributes = $this->ldap->findUserByIdentifier($id);

        if ($attributes === null) {
            return null;
        }

        return new User(
            username: $attributes['uid'] ?? null,
            email: $attributes['mail'] ?? null,
            displayName: $attributes['cn'] ?? null
        );
    }

    public function resolveId(Authenticatable $authenticatable): int|string
    {
        return $authenticatable->email;
    }
}
```

Register custom resolvers via an initializer:

```php
use Tempest\Auth\Authentication\AuthenticatableResolver;

final class LdapAuthenticatableResolverInitializer implements Initializer
{
    #[Singleton]
    public function initialize(Container $container): AuthenticatableResolver
    {
        return new LdapAuthenticatableResolver(
            ldap: $container->get(LdapClient::class),
        );
    }
}
```

## Custom Authenticators

Create custom authenticators by implementing the `Authenticator` interface. Example: request-only authentication:

```php
use Tempest\Auth\Authentication\Authenticator;
use Tempest\Auth\Authentication\Authenticatable;

#[Autowire]
final class RequestAuthenticator implements Authenticator
{
    private ?Authenticatable $current = null;

    public function authenticate(Authenticatable $authenticatable): void
    {
        $this->current = $authenticatable;
    }

    public function deauthenticate(): void
    {
        $this->current = null;
    }

    public function current(): ?Authenticatable
    {
        return $this->current;
    }
}
```

## Access Control & Policies

### Defining Policies

Create policies by adding methods with the `#[Policy]` attribute:

```php
use Tempest\Auth\AccessControl\Policy;
use Tempest\Auth\AccessControl\AccessDecision;

final class PostPolicy
{
    #[Policy(Post::class)]
    public function create(): bool
    {
        return true;
    }

    #[Policy]
    public function view(Post $post): bool
    {
        return $post->published;
    }

    #[Policy(action: ['edit', 'update'])]
    public function edit(Post $post, ?User $user): bool
    {
        if ($user === null) {
            return false;
        }

        return $post->authorId === $user->id->value;
    }
}
```

Return `AccessDecision` for detailed feedback:

```php
return AccessDecision::denied('You must be authenticated to perform this action.');
```

### Checking Permissions

Inject `AccessControl` to verify permissions:

```php
use Tempest\Auth\AccessControl\AccessControl;

final readonly class PostController
{
    public function __construct(
        private AccessControl $accessControl,
    ) {}

    #[Delete('/posts/{post}')]
    public function delete(Post $post): Redirect
    {
        $this->accessControl->ensureGranted('delete', $post);
        // Proceed with deletion...
        return new Redirect('/posts');
    }
}
```

Use `isGranted()` for conditional checks:

```php
$decision = $accessControl->isGranted('delete', $post);
if ($decision->granted) {
    // Allow action
}
```

### Resources Without Instances

Check permissions for resource classes without instances:

```php
$accessControl->isGranted('create', resource: Post::class, subject: $user);
```
