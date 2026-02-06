# Laravel 12.x Release Notes

## Versioning Scheme

Laravel and its first-party packages follow [Semantic Versioning](https://semver.org).

- **Major releases**: Released annually (~Q1)
- **Minor and patch releases**: May be released weekly
- **Breaking changes**: Only in major releases

### Version Constraints

When referencing Laravel, use constraints like `^12.0`:

```php
"laravel/framework": "^12.0"
```

### Named Arguments

Named arguments are **not** covered by Laravel's backwards compatibility guidelines. Function parameter names may change in future versions, so use named arguments cautiously.

## Support Policy

| Version | PHP | Release | Bug Fixes Until | Security Fixes Until |
|---------|-----|---------|-----------------|----------------------|
| 10 | 8.1 - 8.3 | Feb 14, 2023 | Aug 6, 2024 | Feb 4, 2025 |
| 11 | 8.2 - 8.4 | Mar 12, 2024 | Sep 3, 2025 | Mar 12, 2026 |
| 12 | 8.2 - 8.5 | Feb 24, 2025 | Aug 13, 2026 | Feb 24, 2027 |
| 13 | 8.3 - 8.5 | Q1 2026 | Q3 2027 | Q1 2028 |

**Standard Support**: 18 months for bug fixes, 2 years for security fixes

## Laravel 12 Highlights

### Minimal Breaking Changes

Laravel 12 is a **maintenance release** focused on upgrading dependencies with minimal breaking changes. Most applications can upgrade without code modifications.

### New Application Starter Kits

Laravel 12 introduces new starter kits for:

- **React**: Inertia 2, TypeScript, shadcn/ui, Tailwind
- **Vue**: Inertia 2, TypeScript, shadcn/ui, Tailwind
- **Livewire**: Flux UI component library, Laravel Volt, Tailwind

#### Features

All starter kits include:
- Built-in authentication system
- Login & registration
- Password reset
- Email verification

#### WorkOS AuthKit Variant

New WorkOS-powered variants offering:
- Social authentication
- Passkeys
- SSO support
- **Free tier**: Up to 1 million monthly active users

### Deprecations

- **Laravel Breeze**: No longer receiving updates
- **Laravel Jetstream**: No longer receiving updates

For new projects, use the updated [starter kits documentation](../getting-started/starter-kits.md).

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/releases)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
