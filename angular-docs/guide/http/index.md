# HTTP Client Overview
> Source: https://angular.dev/guide/http

Most modern web applications require server communication to download or upload data and access backend services. Angular provides the `HttpClient` service class from `@angular/common/http` to handle these interactions.

## HTTP Client Service Features

The HTTP client service offers four major capabilities:

1. **Typed Response Values** - The ability to request strongly-typed response data
2. **Error Handling** - Streamlined mechanisms for managing request failures
3. **Request/Response Interception** - Capability to intercept and modify HTTP communications
4. **Testing Utilities** - Robust tools for testing HTTP-dependent code

## Technical Details

- **Service Location**: `@angular/common/http`
- **Main Class**: `HttpClient`

## Related Topics

- [Setting up HttpClient](setup.md)
- [Making HTTP requests](making-requests.md)
- [httpResource](http-resource.md)
- [Interceptors](interceptors.md)
- [Testing](testing.md)
