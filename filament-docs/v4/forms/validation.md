# Filament Forms Validation Documentation

## Overview

Filament provides comprehensive validation capabilities for form fields. The framework supports both frontend validation (for immediate user feedback) and backend validation through Laravel's validation system.

**Key principle:** "Validation rules may be added to any field" and Filament includes dedicated validation methods alongside standard Laravel rules.

## Available Validation Rules

Filament offers 49+ dedicated validation methods organized by purpose:

### Date/Time Rules
- `after()` / `afterOrEqual()` - Value must be after a given date
- `before()` / `beforeOrEqual()` - Value must be before a given date

### String Pattern Rules
- `alpha()` - Alphabetic characters only
- `alphaDash()` - Alphanumeric with dashes/underscores
- `alphaNum()` - Alphanumeric only
- `ascii()` - 7-bit ASCII characters only
- `regex()` / `notRegex()` - Match/avoid regex patterns
- `startsWith()` / `endsWith()` - String prefix/suffix validation

### Comparison Rules
- `different()` - Must differ from another field
- `gt()` / `gte()` - Greater than comparisons
- `lt()` / `lte()` - Less than comparisons
- `same()` - Must match another field value

### Database Rules
- `unique()` / `scopedUnique()` - Ensure database uniqueness
- `exists()` / `scopedExists()` - Verify database presence
- The "scoped" variants respect Eloquent global scopes and multi-tenancy

### Format/Type Rules
- `hexColor()` - Valid hex color format
- `ip()` / `ipv4()` / `ipv6()` - IP address validation
- `json()` - Valid JSON string
- `macAddress()` - MAC address format
- `ulid()` / `uuid()` - ULID/UUID formats
- `enum()` - Valid enum values

### Conditional Rules
- `requiredIf()` - Required only when condition met
- `requiredUnless()` - Required unless condition met
- `prohibitedIf()` / `prohibitedUnless()` - Opposite of required rules
- `prohibits()` - Field blocks other fields from having values

### Field State Rules
- `filled()` - Cannot be empty when present
- `nullable()` - Can be empty (default if not required)
- `confirmed()` - Must match `{field}_confirmation`

## Custom & Other Rules

**Using Laravel rules directly:**
```php
TextInput::make('slug')->rules(['alpha_dash'])
```

**Custom validation rule objects:**
```php
TextInput::make('slug')->rules([new Uppercase()])
```

**Closure rules with utility injection:**
```php
TextInput::make('slug')->rules([
    fn (Get $get): Closure => function ($attribute, $value, $fail) use ($get) {
        if ($get('other_field') === 'foo' && $value !== 'bar') {
            $fail("The {$attribute} is invalid.");
        }
    },
])
```

## Configuration Options

### Customizing Validation Attributes
```php
TextInput::make('name')->validationAttribute('full name')
```

Supports dynamic calculation via function with utility injection.

### Custom Error Messages
```php
TextInput::make('email')
    ->unique()
    ->validationMessages([
        'unique' => 'The :attribute has already been registered.',
    ])
```

### HTML in Messages
```php
TextInput::make('password')
    ->allowHtmlValidationMessages()
```

**Warning:** Only use with safe HTML content to prevent XSS attacks.

### Skipping Validation for Unsaved Fields
```php
TextInput::make('name')
    ->required()
    ->saved(false)
    ->validatedWhenNotDehydrated(false)
```

## Key Notes

- Some Laravel validation rules require correct attribute names and won't work via `rule()`—use dedicated Filament methods when available
- Select, toggle button, checkbox list, and radio fields automatically apply `in()` rule based on available options
- Scoped variants (`scopedUnique()`, `scopedExists()`) apply Eloquent global scopes including soft-deletes and multi-tenancy
- The `ignoreRecord` parameter handles update scenarios where the current value shouldn't fail unique validation
