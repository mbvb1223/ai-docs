# Filament Users Documentation - Overview

## Introduction

By default, all `App\Models\User`s can access Filament locally. To enable production access, you must implement authorization checks to ensure only appropriate users gain entry.

## Authorizing Panel Access

Implement the `FilamentUser` contract with a `canAccessPanel()` method:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser
{
    public function canAccessPanel(Panel $panel): bool
    {
        return str_ends_with($this->email, '@yourdomain.com')
            && $this->hasVerifiedEmail();
    }
}
```

You can create panel-specific logic by checking `$panel->getId()` for conditional access rules across multiple panels.

## User Avatars

By default, Filament generates avatars via ui-avatars.com. To customize, implement the `HasAvatar` contract:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasAvatar;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasAvatar
{
    public function getFilamentAvatarUrl(): ?string
    {
        return $this->avatar_url;
    }
}
```

### Custom Avatar Provider Example

Create a provider at `app/Filament/AvatarProviders/BoringAvatarsProvider.php`:

```php
<?php

namespace App\Filament\AvatarProviders;

use Filament\AvatarProviders\Contracts;
use Filament\Facades\Filament;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;

class BoringAvatarsProvider implements Contracts\AvatarProvider
{
    public function get(Model | Authenticatable $record): string
    {
        $name = str(Filament::getNameForDefaultAvatar($record))
            ->trim()
            ->explode(' ')
            ->map(fn (string $segment): string => filled($segment)
                ? mb_substr($segment, 0, 1) : '')
            ->join(' ');

        return 'https://source.boringavatars.com/beam/120/'
            . urlencode($name);
    }
}
```

Register in panel configuration:

```php
use App\Filament\AvatarProviders\BoringAvatarsProvider;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->defaultAvatarProvider(BoringAvatarsProvider::class);
}
```

## User Name Configuration

Implement `HasName` to customize the name attribute display:

```php
<?php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasName;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements FilamentUser, HasName
{
    public function getFilamentName(): string
    {
        return "{$this->first_name} {$this->last_name}";
    }
}
```

## Authentication Features

Enable authentication capabilities in panel configuration:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->login()
        ->registration()
        ->passwordReset()
        ->emailVerification()
        ->emailChangeVerification()
        ->profile();
}
```

### Customizing Authentication Pages

Extend base page classes and override methods:

```php
<?php

namespace App\Filament\Pages\Auth;

use Filament\Auth\Pages\EditProfile as BaseEditProfile;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Schema;

class EditProfile extends BaseEditProfile
{
    public function form(Schema $schema): Schema
    {
        return $schema
            ->components([
                TextInput::make('username')
                    ->required()
                    ->maxLength(255),
                $this->getNameFormComponent(),
                $this->getEmailFormComponent(),
                $this->getPasswordFormComponent(),
                $this->getPasswordConfirmationFormComponent(),
            ]);
    }
}
```

Register the custom page:

```php
use App\Filament\Pages\Auth\EditProfile;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->profile(EditProfile::class);
}
```

### Customizing Individual Fields

Override specific field methods without redefining the entire form:

```php
use Filament\Schemas\Components\Component;

protected function getPasswordFormComponent(): Component
{
    return parent::getPasswordFormComponent()
        ->revealable(false);
}
```

### Email Change Verification

When enabled with the profile feature, users must verify new email addresses before using them. Verification links remain valid for 60 minutes. An email to the old address allows users to block the change.

### Profile Page Sidebar

Enable the standard sidebar layout (unavailable by default due to tenancy compatibility):

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->profile(isSimple: false);
}
```

### Customizing Route Slugs

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->loginRouteSlug('login')
        ->registrationRouteSlug('register')
        ->passwordResetRoutePrefix('password-reset')
        ->passwordResetRequestRouteSlug('request')
        ->passwordResetRouteSlug('reset')
        ->emailVerificationRoutePrefix('email-verification')
        ->emailVerificationPromptRouteSlug('prompt')
        ->emailVerificationRouteSlug('verify')
        ->emailChangeVerificationRoutePrefix('email-change-verification')
        ->emailChangeVerificationRouteSlug('verify');
}
```

### Authentication Guard and Password Broker

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->authGuard('web')
        ->authPasswordBroker('users');
}
```

### Password Input Visibility

Disable the "reveal password" feature globally:

```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->revealablePasswords(false);
}
```

## Guest Access

To enable guest access to a panel:

- Remove `Authenticate::class` from the `authMiddleware()` array
- Remove `->login()` and authentication features
- Remove the default `AccountWidget` from the `widgets()` array

### Guest Authorization in Policies

Make the `User` parameter optional in model policies:

```php
public function viewAny(?User $user): bool
{
    return true;
}
```

Alternatively, remove these methods entirely from the policy.
