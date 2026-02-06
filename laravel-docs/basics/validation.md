# Laravel 12.x Validation Documentation

## Introduction

Laravel provides several different approaches to validate your application's incoming data. The most common approach is using the `validate` method available on all incoming HTTP requests.

## Validation Quickstart

### Writing the Validation Logic

```php
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...

    return redirect('/posts');
}
```

Validation rules can also be specified as arrays:

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

### Displaying Validation Errors

```blade
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### The `@error` Directive

```blade
<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

### Repopulating Forms

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

### Optional Fields

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

## Form Request Validation

### Creating Form Requests

```bash
php artisan make:request StorePostRequest
```

```php
public function rules(): array
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

### Using Form Requests in Controllers

```php
public function store(StorePostRequest $request): RedirectResponse
{
    $validated = $request->validated();

    // Retrieve a portion of the validated input data...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);

    return redirect('/posts');
}
```

### Authorizing Form Requests

```php
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));
    return $comment && $this->user()->can('update', $comment);
}
```

### Customizing Error Messages

```php
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

### Customizing Validation Attributes

```php
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

### Preparing Input for Validation

```php
use Illuminate\Support\Str;

protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

## Manually Creating Validators

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
]);

if ($validator->fails()) {
    return redirect('/post/create')
        ->withErrors($validator)
        ->withInput();
}

$validated = $validator->validated();
```

### Automatic Redirection

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

## Working With Validated Input

```php
$validated = $request->validated();

$validated = $request->safe()->only(['name', 'email']);
$validated = $request->safe()->except(['name', 'email']);
$validated = $request->safe()->all();

// Adding fields to validated data
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);

// Converting to collection
$collection = $request->safe()->collect();
```

## Working With Error Messages

### Retrieving Error Messages

```php
$errors = $validator->errors();

echo $errors->first('email');

foreach ($errors->get('email') as $message) {
    // ...
}

foreach ($errors->all() as $message) {
    // ...
}

if ($errors->has('email')) {
    // ...
}
```

## Available Validation Rules

### Common Rules

- **required** - Field is required
- **nullable** - Field may be null
- **string** - Field must be a string
- **integer** - Field must be an integer
- **numeric** - Field must be numeric
- **boolean** - Field must be boolean
- **array** - Field must be an array
- **date** - Field must be a valid date
- **email** - Field must be a valid email
- **url** - Field must be a valid URL
- **uuid** - Field must be a valid UUID

### Size Rules

- **min:value** - Minimum value/length
- **max:value** - Maximum value/length
- **size:value** - Exact size
- **between:min,max** - Value between min and max

### String Rules

- **alpha** - Alphabetic characters only
- **alpha_num** - Alphanumeric characters only
- **alpha_dash** - Alphanumeric with dashes and underscores
- **lowercase** - Lowercase only
- **uppercase** - Uppercase only
- **regex:pattern** - Must match regex
- **starts_with:foo,bar** - Must start with given values
- **ends_with:foo,bar** - Must end with given values

### Database Rules

- **exists:table,column** - Must exist in database
- **unique:table,column** - Must be unique in database

### File Rules

- **file** - Must be an uploaded file
- **image** - Must be an image
- **mimes:foo,bar** - File must be of given MIME types
- **mimetypes:type** - File must match MIME type
- **extensions:foo,bar** - File must have given extension

### Date Rules

- **after:date** - After given date
- **after_or_equal:date** - After or equal to date
- **before:date** - Before given date
- **before_or_equal:date** - Before or equal to date
- **date_equals:date** - Equal to given date
- **date_format:format** - Must match date format

### Conditional Rules

- **required_if:field,value** - Required if field equals value
- **required_unless:field,value** - Required unless field equals value
- **required_with:foo,bar** - Required if other fields present
- **required_without:foo,bar** - Required if other fields absent

### Comparison Rules

- **same:field** - Must match another field
- **different:field** - Must be different from another field
- **confirmed** - Must have matching confirmation field
- **gt:field** - Greater than
- **gte:field** - Greater than or equal
- **lt:field** - Less than
- **lte:field** - Less than or equal

## Custom Validation Rules

### Using Rule Objects

```php
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

### Using Closures

```php
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        function ($attribute, $value, $fail) {
            if ($value === 'foo') {
                $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/validation)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
