# Laravel 12.x CSRF Protection

## Introduction

Cross-site request forgeries (CSRF) are malicious exploits where unauthorized commands are performed on behalf of an authenticated user. Laravel provides built-in protection against CSRF attacks.

### Explanation of the Vulnerability

A CSRF vulnerability occurs when a malicious website creates a form that submits to your application without the user's knowledge. For example, a form could change a user's email address:

```html
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="[email protected]">
</form>

<script>
    document.forms[0].submit();
</script>
```

Without CSRF protection, if an authenticated user visits the malicious site, their email would be changed without their consent.

## Preventing CSRF Requests

Laravel automatically generates a CSRF "token" for each active user session. This token is stored in the user's session and changes each time the session is regenerated, preventing malicious applications from accessing it.

### Accessing the CSRF Token

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

### Including CSRF Tokens in Forms

For every "POST", "PUT", "PATCH", or "DELETE" HTML form, include a hidden CSRF `_token` field:

```html
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

The `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` middleware (included in the `web` middleware group by default) automatically verifies that the request token matches the session token.

### CSRF Tokens & SPAs

For Single Page Applications (SPAs) using Laravel as an API backend, consult the [Laravel Sanctum documentation](https://laravel.com/docs/12.x/sanctum) for authentication and CSRF protection information.

### Excluding URIs From CSRF Protection

You may exclude specific URIs from CSRF protection in your `bootstrap/app.php` file:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

This is useful for webhook handlers (like Stripe) that cannot provide CSRF tokens.

**Note:** CSRF middleware is automatically disabled for all routes when running tests.

## X-CSRF-TOKEN

The `ValidateCsrfToken` middleware checks for the CSRF token as both a POST parameter and as an `X-CSRF-TOKEN` request header.

### Setup

Store the token in an HTML meta tag:

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Then configure your JavaScript library to automatically include it:

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

This provides CSRF protection for AJAX-based applications using legacy JavaScript.

## X-XSRF-TOKEN

Laravel stores the current CSRF token in an encrypted `XSRF-TOKEN` cookie included with each response. This cookie value can be used to set the `X-XSRF-TOKEN` request header.

Modern JavaScript frameworks like Angular and Axios automatically place the `XSRF-TOKEN` cookie value in the `X-XSRF-TOKEN` header on same-origin requests.

By default, the `resources/js/bootstrap.js` file includes the Axios HTTP library, which automatically sends the `X-XSRF-TOKEN` header.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/csrf)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
