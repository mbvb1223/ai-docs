# Laravel 12.x Precognition Documentation

## Introduction

Laravel Precognition allows you to anticipate the outcome of a future HTTP request. Its primary use case is providing "live" validation for frontend JavaScript applications without duplicating backend validation rules.

## Setup

Add middleware to route:

```php
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

## Using Vue

### Installation

```bash
npm install laravel-precognition-vue
```

### Basic Form

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <input v-model="form.name" @change="form.validate('name')" />
        <div v-if="form.invalid('name')">{{ form.errors.name }}</div>

        <input v-model="form.email" @change="form.validate('email')" />
        <div v-if="form.invalid('email')">{{ form.errors.email }}</div>

        <button :disabled="form.processing">Create User</button>
    </form>
</template>
```

### Form Methods

```javascript
form.setValidationTimeout(3000);
form.forgetError('avatar');
form.reset();
```

### Wizard Pattern

```vue
<button @click="form.validate({
    only: ['name', 'email'],
    onSuccess: () => nextStep(),
})">Next Step</button>
```

## Using React

### Installation

```bash
npm install laravel-precognition-react
```

### Basic Form

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    return (
        <form onSubmit={(e) => { e.preventDefault(); form.submit(); }}>
            <input
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <input
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>Create User</button>
        </form>
    );
}
```

## Using Alpine

### Installation

```bash
npm install laravel-precognition-alpine
```

### Register Plugin

```javascript
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

Alpine.plugin(Precognition);
Alpine.start();
```

### Basic Form

```blade
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    <input x-model="form.name" @change="form.validate('name')" />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <input x-model="form.email" @change="form.validate('email')" />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">Create User</button>
</form>
```

### Repopulating Old Data

```blade
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

## Configuring Axios

```javascript
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

## Validating Arrays

```javascript
form.validate('users.*.email');
form.validate('profile.*');
```

## Customizing Validation Rules

```php
protected function rules()
{
    return [
        'password' => [
            'required',
            $this->isPrecognitive()
                ? Password::min(8)
                : Password::min(8)->uncompromised(),
        ],
    ];
}
```

## Handling File Uploads

Files are not uploaded during precognitive validation by default.

```php
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
        ],
    ];
}
```

Include files in every request:

```javascript
form.validateFiles();
```

## Managing Side-Effects

```php
public function handle(Request $request, Closure $next): mixed
{
    if (! $request->isPrecognitive()) {
        Interaction::incrementFor($request->user());
    }

    return $next($request);
}
```

## Testing

```php
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', ['name' => 'Taylor Otwell']);

    $response->assertSuccessfulPrecognition();
    expect(User::count())->toBe(0);
});
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/precognition)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
