# Laravel 12.x File Storage Documentation

## Introduction

Laravel provides a powerful filesystem abstraction through Flysystem, offering simple drivers for local filesystems, SFTP, and Amazon S3, with a unified API across all storage options.

## Configuration

The filesystem configuration file is located at `config/filesystems.php`. Each disk represents a particular storage driver and storage location.

### The Local Driver

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

### The Public Disk

The `public` disk stores files in `storage/app/public` for publicly accessible files.

**Create symbolic link:**
```bash
php artisan storage:link
```

**Generate URLs:**
```php
echo asset('storage/file.txt');
```

## Driver Prerequisites

### S3 Driver

```bash
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

Set environment variables:
```
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
```

### FTP Driver

```bash
composer require league/flysystem-ftp "^3.0"
```

### SFTP Driver

```bash
composer require league/flysystem-sftp-v3 "^3.0"
```

### Scoped Filesystems

```bash
composer require league/flysystem-path-prefixing "^3.0"
```

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

### Read-Only Filesystems

```bash
composer require league/flysystem-read-only "^3.0"
```

```php
's3-videos' => [
    'driver' => 's3',
    'read-only' => true,
],
```

## Obtaining Disk Instances

```php
use Illuminate\Support\Facades\Storage;

// Default disk
Storage::put('avatars/1', $content);

// Specific disk
Storage::disk('s3')->put('avatars/1', $content);
```

### On-Demand Disks

```php
$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

## Retrieving Files

### Get File Contents

```php
$contents = Storage::get('file.jpg');
$orders = Storage::json('orders.json');
```

### Check File Existence

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // File exists
}

if (Storage::disk('s3')->missing('file.jpg')) {
    // File is missing
}
```

### Downloading Files

```php
return Storage::download('file.jpg');
return Storage::download('file.jpg', $name, $headers);
```

### File URLs

```php
$url = Storage::url('file.jpg');
```

### Temporary URLs

```php
$url = Storage::temporaryUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

### File Metadata

```php
$size = Storage::size('file.jpg');
$time = Storage::lastModified('file.jpg');
$mime = Storage::mimeType('file.jpg');
$path = Storage::path('file.jpg');
```

## Storing Files

### Put Files

```php
Storage::put('file.jpg', $contents);
Storage::put('file.jpg', $resource);
```

### Prepending and Appending

```php
Storage::prepend('file.log', 'Prepended Text');
Storage::append('file.log', 'Appended Text');
```

### Copying and Moving

```php
Storage::copy('old/file.jpg', 'new/file.jpg');
Storage::move('old/file.jpg', 'new/file.jpg');
```

### Automatic Streaming

```php
use Illuminate\Http\File;

$path = Storage::putFile('photos', new File('/path/to/photo'));
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

### File Uploads

```php
$path = $request->file('avatar')->store('avatars');
$path = $request->file('avatar')->storeAs('avatars', $request->user()->id);
$path = $request->file('avatar')->store('avatars', 's3');
```

### File Visibility

```php
Storage::put('file.jpg', $contents, 'public');
$visibility = Storage::getVisibility('file.jpg');
Storage::setVisibility('file.jpg', 'public');

$path = $request->file('avatar')->storePublicly('avatars', 's3');
```

## Deleting Files

```php
Storage::delete('file.jpg');
Storage::delete(['file.jpg', 'file2.jpg']);
Storage::disk('s3')->delete('path/file.jpg');
```

## Directories

### Get All Files

```php
$files = Storage::files($directory);
$files = Storage::allFiles($directory);
```

### Get All Directories

```php
$directories = Storage::directories($directory);
$directories = Storage::allDirectories($directory);
```

### Create a Directory

```php
Storage::makeDirectory($directory);
```

### Delete a Directory

```php
Storage::deleteDirectory($directory);
```

## Testing

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('albums can be uploaded', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
    ]);

    Storage::disk('photos')->assertExists('photo1.jpg');
    Storage::disk('photos')->assertMissing('missing.jpg');
    Storage::disk('photos')->assertCount('/wallpapers', 2);
    Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
});
```

## Custom Filesystems

```php
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

Storage::extend('dropbox', function (Application $app, array $config) {
    $adapter = new DropboxAdapter(new DropboxClient(
        $config['authorization_token']
    ));

    return new FilesystemAdapter(
        new Filesystem($adapter, $config),
        $adapter,
        $config
    );
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/filesystem)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
