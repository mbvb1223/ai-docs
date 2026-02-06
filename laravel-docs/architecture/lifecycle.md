# Request Lifecycle - Laravel 12.x

## Introduction

Understanding how Laravel processes requests helps you develop with more confidence. This document provides a high-level overview of the Laravel framework's request lifecycle, demystifying how the framework operates.

## Lifecycle Overview

### First Steps

The entry point for all requests to a Laravel application is the **`public/index.php`** file. All requests are directed here by your web server (Apache/Nginx) configuration.

The `index.php` file:
1. Loads the Composer-generated autoloader definition
2. Retrieves an instance of the Laravel application from `bootstrap/app.php`
3. Creates an instance of the application/[service container](container.md)

### HTTP / Console Kernels

The incoming request is sent to either:
- **HTTP kernel** (via `handleRequest` method)
- **Console kernel** (via `handleCommand` method)

Both kernels serve as the central location through which all requests flow.

#### HTTP Kernel Responsibilities

The HTTP kernel (`Illuminate\Foundation\Http\Kernel`) defines an array of **bootstrappers** that run before request execution:
- Configure error handling
- Configure logging
- Detect the application environment
- Perform other pre-request tasks

The HTTP kernel also passes the request through the application's **middleware stack**, which handles:
- Reading and writing HTTP sessions
- Detecting maintenance mode
- Verifying CSRF tokens

**Method Signature:**
```
Request → [Kernel] → Response
```

### Service Providers

One of the most important bootstrap actions is loading **service providers** for your application.

#### Service Provider Bootstrap Process

Service providers are responsible for bootstrapping framework components:
- Database
- Queue
- Validation
- Routing

**Execution Flow:**

1. Laravel iterates through the list of providers
2. Instantiates each provider
3. Calls the `register` method on all providers
4. Calls the `boot` method on each provider

> The `boot` method executes after all container bindings are registered and available.

Service providers are the **most important aspect** of the entire Laravel bootstrap process. Every major feature is bootstrapped and configured by a service provider.

**Location:** `bootstrap/providers.php` - Lists user-defined or third-party service providers

### Routing

Once the application is bootstrapped and all service providers registered, the `Request` is handed to the **router** for dispatching.

#### Router Responsibilities

- Dispatches the request to a route or controller
- Runs route-specific middleware

#### Middleware in Action

Middleware provides a mechanism for filtering or examining HTTP requests.

**Example:** Authentication middleware
- If user is **not authenticated** → redirect to login
- If user is **authenticated** → proceed further

Middleware types:
- **Global middleware** (all routes): `PreventRequestsDuringMaintenance`
- **Route-specific middleware** (specific routes/groups)

[Read more: Middleware Documentation](../basics/middleware.md)

#### Request Flow

```
Request → Route Middleware → Route/Controller Method → Response
                              ↓
Response travels back through middleware chain
```

### Finishing Up

Once the route or controller method returns a response:

1. **Response travels outward** through the route's middleware
   - Allows application to modify or examine outgoing response

2. **HTTP kernel returns response** to the application instance's `handleRequest` method

3. **`send` method** called on response object
   - Sends response content to user's web browser

4. **Lifecycle complete** ✓

## Focus on Service Providers

Service providers are **truly the key** to bootstrapping a Laravel application:

1. Application instance is created
2. Service providers are registered
3. Request is handed to the bootstrapped application

### Default Service Provider Configuration

**Location:** `app/Providers` directory

#### AppServiceProvider

- Relatively empty by default
- Great place to add application's own bootstrapping
- Excellent for service container bindings

#### Large Applications

For large applications, create multiple service providers with granular bootstrapping for specific services used by your application.

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/lifecycle)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
