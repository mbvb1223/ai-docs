# Laravel 12.x Helpers Documentation

## Introduction

Laravel includes various global "helper" PHP functions for common tasks.

## Arrays & Objects

### Array Access

```php
use Illuminate\Support\Arr;

// Get value with dot notation
$price = Arr::get($array, 'products.desk.price');
$discount = Arr::get($array, 'products.desk.discount', 0);

// Check existence
Arr::has($array, 'products.desk');
Arr::hasAny($array, ['products.desk', 'products.chair']);

// Collapse arrays
$array = Arr::collapse([[1, 2, 3], [4, 5, 6]]);
// [1, 2, 3, 4, 5, 6]
```

### Array Manipulation

```php
// Pluck values
$names = Arr::pluck($array, 'developer.name');
$names = Arr::pluck($array, 'developer.name', 'developer.id');

// Filter and select
$slice = Arr::only($array, ['name', 'price']);
$slice = Arr::except($array, ['password']);

// First and last
$first = Arr::first($array, fn ($value) => $value >= 150);
$last = Arr::last($array);

// Partition
[$underThree, $aboveThree] = Arr::partition([1, 2, 3, 4, 5], fn ($i) => $i < 3);
```

### Data Functions

```php
// With wildcards
$price = data_get($data, 'products.desk.price');
$names = data_get($data, '*.name');

// Set values
data_set($data, 'products.desk.price', 200);
data_fill($data, 'products.desk.discount', 10);
data_forget($data, 'products.desk.price');
```

## Numbers

```php
use Illuminate\Support\Number;

// Formatting
Number::format(100000);           // 100,000
Number::currency(1000);           // $1,000.00
Number::percentage(10);           // 10%
Number::fileSize(1024);           // 1 KB

// Abbreviation
Number::abbreviate(1000);         // 1K
Number::forHumans(1000);          // 1 thousand

// Spelling
Number::spell(102);               // one hundred and two
Number::ordinal(1);               // 1st

// Utilities
Number::clamp(105, min: 10, max: 100); // 100
```

## Paths

```php
app_path('Http/Controllers');
base_path('vendor/bin');
config_path('app.php');
database_path('factories');
public_path('css/app.css');
resource_path('sass/app.scss');
storage_path('app/file.txt');
lang_path('en/messages.php');
```

## URLs

```php
url('user/profile');
route('route.name', ['id' => 1]);
action([UserController::class, 'index']);
asset('img/photo.jpg');
secure_url('user/profile');
```

## Miscellaneous

### HTTP & Response

```php
abort(403);
abort_if(! Auth::user()->isAdmin(), 403);
back();
redirect('home');
response('Hello World', 200);
```

### Authentication

```php
$user = auth()->user();
$policy = policy(Post::class);
```

### Cache & Config

```php
$value = cache('key', 'default');
cache(['key' => 'value'], 300);

$value = config('app.timezone');
config(['app.debug' => true]);
```

### Encryption & Hashing

```php
$secret = encrypt('my-secret-value');
$value = decrypt($encrypted);
$password = bcrypt('my-password');
```

### Events & Jobs

```php
event(new UserRegistered($user));
broadcast(new UserRegistered($user));
dispatch(new SendEmails);
dispatch_sync(new SendEmails);
```

### Requests & Sessions

```php
$input = request('name');
$value = session('key');
session(['key' => 'value']);
$value = old('name');
```

### Views & Logging

```php
return view('greeting', ['name' => 'Victoria']);
logger()->info('Message');
info('User login', ['id' => $user->id]);
```

### Debugging

```php
dd($value);
dump($value);
report($exception);
```

### Utilities

```php
$collection = collect(['Taylor', 'Abigail']);
blank('');              // true
filled('Laravel');      // true
optional($user)?->name;
tap($user, fn ($u) => Cache::pull('users'));
now();
today();
retry(5, fn () => $api->call(), 100);
rescue(fn () => risky(), 'default');
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/helpers)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
