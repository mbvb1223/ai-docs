# Angular Security Guide
> Source: https://angular.dev/best-practices/security

## Overview

This comprehensive guide covers Angular's built-in protections against common web vulnerabilities including cross-site scripting (XSS), cross-site request forgery (CSRF), and cross-site script inclusion (XSSI). It does not address application-level concerns like authentication or authorization.

## Reporting Security Vulnerabilities

Angular participates in Google's "Open Source Software Vulnerability Reward Program." Security researchers should submit vulnerability reports to [https://bughunters.google.com](https://bughunters.google.com/report) rather than public channels.

For additional context on Google's security practices, see [Google's security philosophy](https://www.google.com/about/appsecurity).

## Best Practices

### 1. Maintain Current Library Versions
Keep Angular libraries updated to receive security patches. Review the [Angular changelog](https://github.com/angular/angular/blob/main/CHANGELOG.md) for security-related fixes.

### 2. Avoid Custom Angular Modifications
Customized Angular versions may lag behind current releases and miss critical security updates. Contribute improvements back to the community instead.

### 3. Avoid Security-Risk APIs
Angular documents certain APIs as security sensitive. Refer to the "Trusting safe values" section for details on these high-risk methods.

---

## Preventing Cross-Site Scripting (XSS)

### What is XSS?

Cross-site scripting enables attackers to inject malicious code into web pages that can steal credentials, impersonate users, or execute unauthorized actions. It ranks among the most prevalent web vulnerabilities.

Attackers exploit DOM entry points beyond simple `<script>` tags -- events like `onerror` on images and `javascript:` URLs in links also enable code execution.

### Angular's XSS Security Model

**Default Untrusted Treatment:** Angular assumes all values are untrusted. During template binding and interpolation, it sanitizes and escapes values automatically.

**Trusted Templates:** Angular templates themselves are considered trusted executable code by design. Never construct templates by concatenating user input with template syntax, as this enables code injection.

**AOT Compilation Required:** Always deploy with Ahead-Of-Time (AOT) compilation in production to prevent template injection vulnerabilities.

**Defense-in-Depth:** Content Security Policy (CSP) and Trusted Types provide additional browser-level protections that prevent bypass via lower-level APIs.

### Sanitization and Security Contexts

Sanitization inspects untrusted values and converts them to DOM-safe equivalents. The process depends on context -- safe CSS might be dangerous in URLs.

#### Security Contexts in Angular

| Context | Usage | Details |
|---------|-------|---------|
| **HTML** | `[innerHTML]` binding | Removes malicious scripts but preserves safe tags |
| **Style** | CSS property binding | Validates style values |
| **URL** | `[href]`, `[src]` | Prevents dangerous protocols like `javascript:` |
| **Resource URL** | `<script src>`, `<iframe src>` | Cannot be sanitized -- requires trusted sources only |

Angular sanitizes HTML and URLs but cannot sanitize resource URLs (they contain executable code). Development mode logs sanitization warnings.

### Sanitization Example

```html
<h3>Binding innerHTML</h3>
<p>Bound value:</p>
<p class="e2e-inner-html-interpolated">{{ htmlSnippet }}</p>
<p>Result of binding to innerHTML:</p>
<p class="e2e-inner-html-bound" [innerHTML]="htmlSnippet"></p>
```

```typescript
export class InnerHtmlBindingComponent {
  htmlSnippet = 'Template <script>alert("0wned")</script> <b>Syntax</b>';
}
```

**Behavior:**
- **Interpolation:** Always escapes HTML. The `<script>` tag displays as literal text.
- **innerHTML binding:** Angular sanitizes by removing `<script>` but preserves safe elements like `<b>`.

### Direct DOM API Usage and Explicit Sanitization

Browser DOM APIs and many third-party libraries lack automatic XSS protections. Direct DOM manipulation circumvents Angular's safety mechanisms.

**When unavoidable:** Use `DomSanitizer.sanitize()` with the appropriate `SecurityContext`:

```typescript
import { DomSanitizer, SecurityContext } from '@angular/core';

constructor(private sanitizer: DomSanitizer) {}

sanitizeUrl(url: string): string {
  return this.sanitizer.sanitize(SecurityContext.URL, url);
}
```

### Trusting Safe Values

When applications legitimately need executable code, iframes from trusted sources, or constructed URLs, explicitly mark values as trusted to bypass sanitization.

**Methods:**
- `bypassSecurityTrustHtml`
- `bypassSecurityTrustScript`
- `bypassSecurityTrustStyle`
- `bypassSecurityTrustUrl`
- `bypassSecurityTrustResourceUrl`

**Important:** Only trust values whose safety you have verified. Misuse introduces vulnerabilities.

#### Trust URL Example

```html
<h4>An untrusted URL:</h4>
<p><a class="e2e-dangerous-url" [href]="dangerousUrl">Click me</a></p>
<h4>A trusted URL:</h4>
<p><a class="e2e-trusted-url" [href]="trustedUrl">Click me</a></p>
```

```typescript
private sanitizer = inject(DomSanitizer);

constructor() {
  this.dangerousUrl = 'javascript:alert("Hi there")';
  this.trustedUrl = this.sanitizer.bypassSecurityTrustUrl(this.dangerousUrl);
}
```

Angular sanitizes the first link (disabling the `javascript:` protocol). The second trusts the value explicitly.

#### Trust Resource URL (iframe) Example

```html
<h4>Resource URL:</h4>
<p>Showing: {{ dangerousVideoUrl }}</p>
<p>Trusted:</p>
<iframe
  class="e2e-iframe-trusted-src"
  width="640"
  height="390"
  [src]="videoUrl"
  title="trusted video url">
</iframe>
<p>Untrusted:</p>
<iframe
  class="e2e-iframe-untrusted-src"
  width="640"
  height="390"
  [src]="dangerousVideoUrl"
  title="unTrusted video url">
</iframe>
```

```typescript
updateVideoUrl(id: string) {
  this.dangerousVideoUrl = 'https://www.youtube.com/embed/' + id;
  this.videoUrl = this.sanitizer.bypassSecurityTrustResourceUrl(
    this.dangerousVideoUrl
  );
}
```

Constructing URLs near the input point makes safety verification simpler.

### Content Security Policy (CSP)

CSP is a defense-in-depth technique preventing XSS execution. Configure your web server to return appropriate HTTP headers.

#### Minimal Policy for Angular Apps

```
default-src 'self';
style-src 'self' 'nonce-randomNonceGoesHere';
script-src 'self' 'nonce-randomNonceGoesHere';
```

**Nonce Configuration:**

1. **Workspace Configuration:** Set `autoCsp: true` in `angular.json`
2. **HTML Attribute:** Add `ngCspNonce="randomNonce"` to the root element
3. **Injection Token:** Use the `CSP_NONCE` provider at bootstrap

```typescript
import { bootstrapApplication, CSP_NONCE } from '@angular/core';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: CSP_NONCE,
      useValue: globalThis.myRandomNonceValue,
    },
  ],
});
```

**Critical:** Nonces must be unique per request and unpredictable. If attackers can guess nonces, CSP protection fails.

**Inline CSS Limitation:** Critical CSS inlining cannot use the `CSP_NONCE` token; prefer `autoCsp` or the `ngCspNonce` attribute instead.

#### Policy Components

| Section | Purpose |
|---------|---------|
| `default-src 'self';` | Load all resources from the same origin |
| `style-src 'self' 'nonce-...';` | Allow global styles from same origin and nonce-protected Angular styles |
| `script-src 'self' 'nonce-...';` | Allow scripts from same origin and nonce-protected Angular CLI scripts (for critical CSS only) |

Expand these rules as your application grows and requires additional third-party resources.

### Enforcing Trusted Types

Trusted Types enforce browser-level constraints against DOM XSS by requiring all DOM-manipulating operations to use certified safe objects.

#### Browser Support

Check [caniuse.com/trusted-types](https://caniuse.com/trusted-types) for current support. In unsupported browsers, Angular's `DomSanitizer` continues protecting against XSS.

#### Angular Policies

| Policy | Purpose |
|--------|---------|
| `angular` | Angular internal code; required for framework operation |
| `angular#bundler` | Angular CLI lazy chunk generation |
| `angular#unsafe-bypass` | Required if using `DomSanitizer` bypass methods |
| `angular#unsafe-jit` | Required for Just-In-Time compilation |
| `angular#unsafe-upgrade` | Required for AngularJS hybrid applications |

#### Header Configuration Locations

- Production serving infrastructure
- Angular CLI (`ng serve`) via `headers` in `angular.json`
- Karma (`ng test`) via `customHeaders` in `karma.config.js`

#### Example Headers

**Basic Trusted Types:**
```
Content-Security-Policy: trusted-types angular; require-trusted-types-for 'script';
```

**With Bypass Methods:**
```
Content-Security-Policy: trusted-types angular angular#unsafe-bypass; require-trusted-types-for 'script';
```

**With JIT:**
```
Content-Security-Policy: trusted-types angular angular#unsafe-jit; require-trusted-types-for 'script';
```

**With Lazy Loading:**
```
Content-Security-Policy: trusted-types angular angular#bundler; require-trusted-types-for 'script';
```

### Using the AOT Template Compiler

The Ahead-Of-Time (AOT) compiler prevents template injection vulnerabilities and improves performance. It is the default in Angular CLI and mandatory for production.

**Why not JIT:** The Just-In-Time compiler evaluates templates as code at runtime. Dynamically generating templates -- especially with user data -- bypasses Angular's protections and is a security anti-pattern.

### Server-Side XSS Protection

HTML constructed server-side is vulnerable to template injection. Never generate Angular templates on the server using server-side templating languages; this introduces template-injection vulnerabilities.

Use server templating for non-Angular output only, and ensure it escapes values automatically.

---

## HTTP-Level Vulnerabilities

Angular provides built-in support for two common HTTP attacks, though server-side mitigation is primary.

### Cross-Site Request Forgery (CSRF/XSRF)

#### Attack Vector

An attacker tricks a logged-in user into visiting a malicious page (e.g., `evil.com`). That page sends a forged request to the user's trusted application (e.g., `example-bank.com`). The browser automatically includes authentication cookies, making the forged request appear legitimate.

#### Prevention Mechanism

The application and server cooperate using a token-based anti-XSRF technique:

1. **Server** sends a randomly generated authentication token in a cookie
2. **Client** reads the token from the cookie and includes it in a custom HTTP header on all requests
3. **Server** compares the cookie value to the header value
4. **Same-origin policy** ensures only code from your domain can read the cookie and set custom headers

Malicious code on `evil.com` cannot access cookies or set headers for `example-bank.com`.

### HttpClient XSRF/CSRF Security

`HttpClient` automatically implements XSRF protection. An interceptor reads the `XSRF-TOKEN` cookie and adds it as an `X-XSRF-TOKEN` header on mutating requests (POST, PUT, DELETE, PATCH) to relative and same-origin URLs.

**GET requests are not protected** because the same-origin policy already prevents cross-origin GET requests from accessing responses.

#### Server Requirements

Your server must:
1. Set the `XSRF-TOKEN` cookie on page load or the first GET request
2. Verify that the `X-XSRF-TOKEN` header matches the cookie value
3. Reject requests where token and header do not match

The token should be unique per user and verifiable by the server (e.g., a digest of the session cookie with a salt).

In multi-app environments, use unique cookie names per application to prevent collisions.

### Configure Custom Cookie/Header Names

If your backend uses different names, override defaults with `withXsrfConfiguration`:

```typescript
import { provideHttpClient, withXsrfConfiguration } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withXsrfConfiguration({
        cookieName: 'CUSTOM_XSRF_TOKEN',
        headerName: 'X-Custom-Xsrf-Header',
      }),
    ),
  ],
};
```

### Disabling XSRF Protection

If necessary, disable protection using `withNoXsrfProtection`:

```typescript
import { provideHttpClient, withNoXsrfProtection } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withNoXsrfProtection())],
};
```

#### Additional Resources

- OWASP: [Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)
- OWASP: [CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- Stanford University: [Robust Defenses for Cross-Site Request Forgery](https://seclab.stanford.edu/websec/csrf/csrf.pdf)
- Dave Smith's AngularConnect 2016 talk on XSRF

### Cross-Site Script Inclusion (XSSI)

Also called "JSON vulnerability," XSSI allows attackers' websites to read JSON API data by overriding JavaScript object constructors and including API URLs via `<script>` tags on older browsers.

#### Server-Side Mitigation

Servers prevent this by prefixing all JSON responses with a non-executable string, by convention: `")]}',\n"`

#### Angular's Automatic Protection

`HttpClient` recognizes this convention and automatically strips `")]}',\n"` from all responses before parsing.

See the [Google web security blog](https://security.googleblog.com/2011/05/website-security-for-webmasters.html) for additional details.

---

## Auditing Angular Applications

Angular applications require the same security review practices as standard web applications. Pay particular attention to security-sensitive APIs documented in Angular's reference, especially methods like `bypassSecurityTrust*`, which are explicitly marked as security risks.

---

## Additional Resources

- **OWASP Guide:** [Open Web Application Security Project](https://www.owasp.org/index.php/Category:OWASP_Guide_Project)
- **Trusted Types:** [W3C Trusted Types Specification](https://w3c.github.io/trusted-types/dist/spec/)
- **CSP Guide:** [Web Fundamentals Content Security Policy](https://developers.google.com/web/fundamentals/security/csp)
- **Angular Reference:** DomSanitizer API, SecurityContext, CSP_NONCE token
