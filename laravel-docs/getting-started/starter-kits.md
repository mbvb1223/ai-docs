# Laravel 12.x Starter Kits Documentation

## Introduction

Laravel offers application starter kits that provide a head start when building new Laravel applications. These kits include pre-configured routes, controllers, and views for user registration and authentication using [Laravel Fortify](../packages/fortify.md).

While starter kits are convenient, they are optional. You can build your application from scratch if preferred.

## Creating an Application Using a Starter Kit

### Prerequisites
First, install PHP and the Laravel CLI tool:

```bash
composer global require laravel/installer
```

### Creating a New Application

Create a new Laravel application with:

```bash
laravel new my-app
```

The Laravel installer will prompt you to select your preferred starter kit.

### Starting Development

After creating your application:

```bash
cd my-app
npm install && npm run build
composer run dev
```

Your application will be accessible at `http://localhost:8000`.

## Available Starter Kits

### React

- **Framework**: Inertia with React 19
- **Styling**: Tailwind CSS 4
- **Components**: shadcn/ui component library
- **Features**: Modern SPA with server-side routing

### Vue

- **Framework**: Inertia with Vue 3 Composition API
- **Styling**: Tailwind CSS
- **Components**: shadcn-vue component library
- **Features**: Single-page Vue applications with classic routing

### Livewire

- **Framework**: Laravel Livewire
- **Styling**: Tailwind CSS
- **Components**: Flux UI component library
- **Features**: Dynamic reactive UIs using PHP only

## Starter Kit Customization

### React Customization

**Directory Structure**:
```
resources/js/
├── components/    # Reusable React components
├── hooks/         # React hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

**Adding Components**:
```bash
npx shadcn@latest add switch
```

**Usage Example**:
```tsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

**Layout Options**:
- **Sidebar Layout** (default): `@/layouts/app/app-sidebar-layout`
- **Header Layout**: `@/layouts/app/app-header-layout`

**Sidebar Variants**:
```tsx
<Sidebar collapsible="icon" variant="sidebar">  // Default
<Sidebar collapsible="icon" variant="inset">    // Inset
// Floating variant also available
```

**Authentication Layout Variants**: `simple`, `card`, `split`

### Vue Customization

**Directory Structure**:
```
resources/js/
├── components/    # Reusable Vue components
├── composables/   # Vue composables / hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

**Adding Components**:
```bash
npx shadcn-vue@latest add switch
```

**Usage Example**:
```vue
<script setup lang="ts">
import { Switch } from '@/Components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

**Layout Options**:
- **Sidebar Layout** (default): `@/layouts/app/AppSidebarLayout.vue`
- **Header Layout**: `@/layouts/app/AppHeaderLayout.vue`

### Livewire Customization

**Directory Structure**:
```
resources/views/
├── components            # Reusable components
├── flux                  # Customized Flux components
├── layouts               # Application layouts
├── pages                 # Livewire pages
├── partials              # Reusable Blade partials
├── dashboard.blade.php   # Authenticated user dashboard
└── welcome.blade.php     # Guest user welcome page
```

**Switching to Header Layout**:
```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

**Authentication Layout Variants**: `simple`, `card`, `split`

## Authentication

All starter kits use [Laravel Fortify](../packages/fortify.md) for authentication.

### Fortify Routes

| Route | Method | Description |
|-------|--------|-------------|
| `/login` | GET | Display login form |
| `/login` | POST | Authenticate user |
| `/logout` | POST | Log user out |
| `/register` | GET | Display registration form |
| `/register` | POST | Create new user |
| `/forgot-password` | GET | Display password reset request form |
| `/forgot-password` | POST | Send password reset link |
| `/reset-password/{token}` | GET | Display password reset form |
| `/reset-password` | POST | Update password |
| `/email/verify` | GET | Display email verification notice |
| `/email/verify/{id}/{hash}` | GET | Verify email address |
| `/email/verification-notification` | POST | Resend verification email |
| `/user/confirm-password` | GET | Display password confirmation form |
| `/user/confirm-password` | POST | Confirm password |
| `/two-factor-challenge` | GET | Display 2FA challenge form |
| `/two-factor-challenge` | POST | Verify 2FA code |

### Enabling and Disabling Features

Configure features in `config/fortify.php`:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

Comment out or remove features to disable them.

### Customizing User Creation and Password Reset

Modify action classes in `app/Actions/Fortify/`:

**File**: `CreateNewUser.php`
```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'],
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'],
        'password' => Hash::make($input['password']),
    ]);
}
```

### Two-Factor Authentication

2FA is enabled by default via `Features::twoFactorAuthentication()` in `config/fortify.php`.

Options:
- `confirm`: Requires users to verify a code before 2FA is fully enabled
- `confirmPassword`: Requires password confirmation before enabling/disabling 2FA

### Rate Limiting

Customize rate limiting in your `FortifyServiceProvider`:

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

## WorkOS AuthKit Authentication

Alternative authentication provider offering:
- Social authentication (Google, Microsoft, GitHub, Apple)
- Passkey authentication
- Email-based "Magic Auth"
- SSO

### Configuration

Set environment variables:

```env
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

### Recommendations

- Disable "Email + Password" authentication in WorkOS AuthKit settings
- Configure session inactivity timeout to match Laravel's session timeout (typically 2 hours)

## Inertia SSR

Build an SSR-compatible bundle:

```bash
npm run build:ssr
```

For convenience, use:

```bash
composer dev:ssr
```

This starts both the Laravel development server and Inertia SSR server.

## Community Maintained Starter Kits

Create applications using community starter kits:

```bash
laravel new my-app --using=example/starter-kit
```

### Publishing Community Starter Kits

To publish a starter kit to Packagist:
1. Define required environment variables in `.env.example`
2. List post-installation commands in the `post-create-project-cmd` array of `composer.json`

## Frequently Asked Questions

### How do I upgrade?

No need to update the starter kit itself. Once created, you own the code and can customize it as needed.

### How do I enable email verification?

Implement `MustVerifyEmail` interface in your User model:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

Add the `verified` middleware to protected routes:

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

*Note: Email verification is not required when using WorkOS starter kits.*

### How do I modify the default email template?

Publish email views:

```bash
php artisan vendor:publish --tag=laravel-mail
```

Customize files in `resources/views/vendor/mail/`, including the CSS in `resources/views/vendor/mail/themes/default.css`.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/starter-kits)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
