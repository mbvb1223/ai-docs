# File Storage

## Overview

Tempest provides a unified interface for interacting with multiple filesystem types, including local storage, Amazon S3, Cloudflare R2, and FTP servers. The implementation leverages Flysystem, a battle-tested abstraction layer for file systems.

## Getting Started

### Configuration

Create a configuration file for your desired filesystem provider. For Amazon S3:

```php
// app/s3.config.php
return new S3StorageConfig(
    bucket: env('S3_BUCKET'),
    region: env('S3_REGION'),
    accessKeyId: env('S3_ACCESS_KEY_ID'),
    secretAccessKey: env('S3_SECRET_ACCESS_KEY'),
);
```

### Basic Usage

Access the storage interface through dependency injection:

```php
final readonly class UserService
{
    public function __construct(
        private Storage $storage,
    ) {}

    public function getProfilePictureUrl(User $user): string
    {
        return $this->storage->publicUrl($user->profile_picture_path);
    }
}
```

## Storage Interface Methods

- `publicUrl($location)` — Retrieves a public URL to a file
- `write($location, $contents)` — Writes content to a specified location
- `read($location)` — Reads file contents
- `delete($location)` — Removes files or directories
- `fileOrDirectoryExists($location)` — Checks file/directory existence

## Supported Providers

- LocalStorageConfig
- R2StorageConfig
- S3StorageConfig
- AzureStorageConfig
- FTPStorageConfig
- GoogleCloudStorageConfig
- InMemoryStorageConfig
- SFTPStorageConfig
- ZipArchiveStorageConfig
- CustomStorageConfig

## Multiple Storages

Configure multiple storage locations using tags:

```php
// app/userdata.storage.config.php
return new S3StorageConfig(
    tag: StorageLocation::USER_DATA,
    bucket: env('USERDATA_S3_BUCKET'),
    // ...
);
```

Inject tagged storage:

```php
final readonly class BackupService
{
    public function __construct(
        #[Tag(StorageLocation::BACKUPS)]
        private Storage $storage,
    ) {}
}
```

## Read-Only Storage

Restrict write operations by installing the read-only adapter and setting `readonly: true`:

```php
composer require league/flysystem-read-only
```

## Custom Storage

Implement a custom adapter by creating a `CustomStorageConfig`:

```php
return new CustomStorageConfig(
    adapter: App\MyCustomFilesystemAdapter::class,
);
```

## Testing

### Faking Storage

Replace the storage with a testing implementation:

```php
$storage = $this->storage->fake();
$storage->assertFileExists('file.txt');
```

Fake storages are stored in `.tempest/tests/storage` and cleared on each call.

### Preventing Unauthorized Access

Prevent unintended storage access during tests:

```php
$this->storage->preventUsageWithoutFake();
```
