# File Upload - Forms - Filament Documentation

## Introduction

The file upload field in Filament is built on [Filepond](https://pqina.nl/filepond). Basic usage is straightforward:

```php
use Filament\Forms\Components\FileUpload;

FileUpload::make('attachment')
```

> "Filament also supports `spatie/laravel-medialibrary`"

## Storage Configuration

### Disk and Directory Setup

Files upload to the default storage disk from your configuration. Customize this per field:

```php
FileUpload::make('attachment')
    ->disk('s3')
    ->directory('form-attachments')
    ->visibility('public')
```

Both `disk()`, `directory()`, and `visibility()` accept static values or dynamic functions supporting utility injection including `$get`, `$livewire`, `$record`, and `$state`.

## Multiple File Uploads

Enable multiple file uploads with JSON storage:

```php
FileUpload::make('attachments')
    ->multiple()
```

Add an `array` cast to your Eloquent model property:

```php
protected function casts(): array
{
    return [
        'attachments' => 'array',
    ];
}
```

### Controlling Parallel Uploads

Limit simultaneous uploads using `maxParallelUploads()`:

```php
FileUpload::make('attachments')
    ->multiple()
    ->maxParallelUploads(1)
```

## File Name Control

> "Before using the `preserveFilenames()` or `getUploadedFileNameForStorageUsing()` methods, please be aware of the security implications."

### Security Considerations

Using these methods with local/public disks risks remote code execution if users upload deceptively-named PHP files. S3 disks provide protection against this vector. Consider storing original names separately while keeping random file names in storage.

### Preserving Original Names

```php
FileUpload::make('attachment')
    ->preserveFilenames()
```

### Custom File Names

```php
use Livewire\Features\SupportFileUploads\TemporaryUploadedFile;

FileUpload::make('attachment')
    ->getUploadedFileNameForStorageUsing(
        fn (TemporaryUploadedFile $file): string => (string) str($file->getClientOriginalName())
            ->prepend('custom-prefix-'),
    )
```

### Storing Original Names Independently

```php
FileUpload::make('attachments')
    ->multiple()
    ->storeFileNamesIn('attachment_file_names')
```

## Avatar Mode

Display uploaded images in a compact circle layout:

```php
FileUpload::make('avatar')
    ->avatar()
```

Works well with the circle cropper feature.

## Image Editing

### Basic Image Editor

```php
FileUpload::make('image')
    ->image()
    ->imageEditor()
```

Opens via pencil icon after upload.

### Aspect Ratio Options

```php
FileUpload::make('image')
    ->image()
    ->imageEditor()
    ->imageEditorAspectRatioOptions([
        null,  // Free cropping
        '16:9',
        '4:3',
        '1:1',
    ])
```

### Editor Configuration

```php
FileUpload::make('image')
    ->image()
    ->imageEditor()
    ->imageEditorMode(2)
    ->imageEditorEmptyFillColor('#000000')
    ->imageEditorViewportWidth('1920')
    ->imageEditorViewportHeight('1080')
```

### Circle Cropper

```php
FileUpload::make('image')
    ->image()
    ->avatar()
    ->imageEditor()
    ->circleCropper()
```

### Automatic Aspect Ratio Enforcement

```php
FileUpload::make('banner')
    ->image()
    ->imageAspectRatio('16:9')
    ->automaticallyOpenImageEditorForAspectRatio()
```

Combines aspect ratio validation with automatic editor opening when needed.

### Automatic Image Resizing

```php
FileUpload::make('image')
    ->image()
    ->automaticallyCropImagesToAspectRatio('16:9')
    ->automaticallyResizeImagesMode('cover')
    ->automaticallyResizeImagesToWidth('1920')
    ->automaticallyResizeImagesToHeight('1080')
```

## Appearance Customization

Alter the Filepond component's visual presentation:

```php
FileUpload::make('attachment')
    ->imagePreviewHeight('250')
    ->loadingIndicatorPosition('left')
    ->panelAspectRatio('2:1')
    ->panelLayout('integrated')
    ->removeUploadedFileButtonPosition('right')
    ->uploadButtonPosition('left')
    ->uploadProgressIndicatorPosition('left')
```

### Grid Layout

```php
FileUpload::make('attachments')
    ->multiple()
    ->panelLayout('grid')
```

## File Management Features

### Reordering Files

```php
FileUpload::make('attachments')
    ->multiple()
    ->reorderable()
    ->appendFiles()  // Append new uploads to end
```

### Opening Files

Add buttons to open files in new tabs:

```php
FileUpload::make('attachments')
    ->multiple()
    ->openable()
```

### Downloading Files

Enable download functionality:

```php
FileUpload::make('attachments')
    ->multiple()
    ->downloadable()
```

### Preview Control

Disable file previews:

```php
FileUpload::make('attachments')
    ->multiple()
    ->previewable(false)
```

## Storage Options

### Moving vs. Copying

```php
FileUpload::make('attachment')
    ->moveFiles()
```

Moves files instead of copying when forms submit (requires same disk for temporary and permanent storage).

### Temporary-Only Uploads

```php
FileUpload::make('attachment')
    ->storeFiles(false)
```

Returns temporary file upload objects instead of permanent paths—useful for imports.

## Additional Configuration

### EXIF Orientation

```php
FileUpload::make('attachment')
    ->orientImagesFromExif(false)
```

### Delete Button

Hide the remove button:

```php
FileUpload::make('attachment')
    ->deletable(false)
```

### Clipboard Pasting

```php
FileUpload::make('attachment')
    ->pasteable(false)
```

### File Information Fetching

```php
FileUpload::make('attachment')
    ->fetchFileInformation(false)
```

Improves performance with remote storage and many files.

### Upload Message

```php
FileUpload::make('attachment')
    ->uploadingMessage('Uploading attachment...')
```

## Validation

### File Type Restrictions

```php
FileUpload::make('document')
    ->acceptedFileTypes(['application/pdf'])
```

For images:

```php
FileUpload::make('image')
    ->image()
```

#### Custom MIME Type Mapping

```php
FileUpload::make('designs')
    ->acceptedFileTypes([
        'x-world/x-3dmf',
        'application/vnd.sketchup.skp',
    ])
    ->mimeTypeMap([
        '3dm' => 'x-world/x-3dmf',
        'skp' => 'application/vnd.sketchup.skp',
    ])
```

### File Size Validation

```php
FileUpload::make('attachment')
    ->minSize(512)      // KB
    ->maxSize(1024)     // KB
```

#### Large File Uploads

Update `php.ini`:

```ini
post_max_size = 120M
upload_max_filesize = 120M
```

Configure Livewire (`config/livewire.php`):

```php
'temporary_file_upload' => [
    'rules' => ['required', 'file', 'max:122880'],  // 120MB in KB
],
```

### Image Dimension Validation

```php
use Illuminate\Validation\Rule;

FileUpload::make('photo')
    ->image()
    ->rule(
        Rule::dimensions()
            ->minWidth(800)
            ->minHeight(600)
            ->maxWidth(1920)
            ->maxHeight(1080)
    )
```

### Image Aspect Ratio Validation

```php
FileUpload::make('banner')
    ->image()
    ->imageAspectRatio('16:9')
```

Multiple ratios:

```php
FileUpload::make('banner')
    ->image()
    ->imageAspectRatio(['16:9', '4:3', '1:1'])
```

Or with ratio ranges:

```php
FileUpload::make('banner')
    ->image()
    ->rule(Rule::dimensions()->minRatio(4/3)->maxRatio(16/9))
```

> "These dimension and aspect ratio validation rules only apply to newly uploaded files."
