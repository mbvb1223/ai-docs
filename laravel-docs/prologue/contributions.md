# Laravel 12.x Contribution Guide

## Overview
This guide outlines how to contribute to Laravel, covering bug reports, pull requests, coding standards, and community guidelines.

## Bug Reports

**Key Points:**
- Laravel strongly encourages **pull requests** over bug reports
- Pull requests must be marked "ready for review" (not draft) with all tests passing
- Draft pull requests are closed after a few days of inactivity

**When Filing a Bug Report, include:**
- Clear title and description
- Relevant information and code samples demonstrating the issue
- The goal is to make reproduction easy for others

**Important Notes:**
- Don't create GitHub issues for improper DocBlocks, PHPStan, or IDE warnings—submit a pull request instead
- You can help by fixing bugs listed in [Laravel's issue trackers](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel)

### Laravel Repositories
Core repositories include:
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Passport](https://github.com/laravel/passport)
- And 20+ other official packages

## Support Questions

GitHub issue trackers are **not** for support. Use these channels instead:
- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

## Core Development Discussion

- Propose features on the [Laravel framework GitHub discussions](https://github.com/laravel/framework/discussions)
- Be willing to implement at least some code for proposed features
- Discuss informally in the `#internals` channel of [Laravel Discord](https://discord.gg/laravel)
- Taylor Otwell (maintainer) is typically available weekdays 8am-5pm UTC-06:00

## Which Branch?

| Type | Target Branch | Notes |
|------|---------------|-------|
| **Bug Fixes** | `12.x` (latest) | Never send to `master` unless fixing upcoming-only features |
| **Minor Features** | `12.x` | Must be fully backward compatible |
| **Major Features** | `master` | For features with breaking changes or major additions |

## Compiled Assets

**Critical Rule:** Do NOT commit compiled files (CSS, JS in `resources/` directories)

- Compiled files cannot be realistically reviewed
- Could be exploited to inject malicious code
- Laravel maintainers will generate and commit compiled files defensively

## Security Vulnerabilities

If you discover a security vulnerability, email Taylor Otwell at **taylor@laravel.com** directly. Do not use public issue trackers. All vulnerabilities are promptly addressed.

## Coding Style

Laravel follows:
- **PSR-2** coding standard
- **PSR-4** autoloading standard

### PHPDoc

**Format Example:**
```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

**Rules:**
- Two spaces after `@param`, type, two more spaces, then variable name
- If using native types, `@param`/`@return` can be removed when redundant
- For generic types, use `@param` or `@return` to specify generics:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

### StyleCI

[StyleCI](https://styleci.io/) automatically fixes code style after pull requests merge, so focus on content rather than style.

## Code of Conduct

Derived from the Ruby Code of Conduct. Report violations to Taylor Otwell at **taylor@laravel.com**

**Expected Behavior:**
- ✅ Tolerate opposing views
- ✅ Keep language and actions free of personal attacks
- ✅ Assume good intentions when interpreting others' words and actions
- ❌ Harassment will not be tolerated

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/contributions)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
