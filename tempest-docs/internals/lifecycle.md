# Framework Lifecycle

## Booting

The Tempest framework initializes through an entry point, typically `public/index.php` or `./tempest`. The HTTP context uses `HttpApplication`, while console operations use `ConsoleApplication`.

During startup, the `FrameworkKernel` performs several key operations:

- Loads environment settings, configures exception handling, and sets up the container
- Executes discovery via `LoadDiscoveryLocations` and `LoadDiscoveryClasses` classes
- Registers configuration files through the `LoadConfig` class

When bootstrapping is completed, the `Tempest\Core\KernelEvent::BOOTED` event is fired.

## Shutdown

The shutdown sequence runs through the kernel's `shutdown()` method, invoked at the conclusion of both HTTP and console operations, as well as within exception handlers. This process:

- Executes deferred tasks
- Dispatches the `KernelEvent::SHUTDOWN` event
- Handles cleanup operations
- Terminates the application process
