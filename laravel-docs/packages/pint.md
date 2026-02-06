# Laravel 12.x Pint Documentation

## Introduction

Laravel Pint is an opinionated PHP code style fixer built on PHP CS Fixer.

## Installation

```bash
composer require laravel/pint --dev
```

## Running Pint

```bash
./vendor/bin/pint
```

### Common Options

```bash
./vendor/bin/pint -v                  # Verbose output
./vendor/bin/pint --test              # Test mode (no changes)
./vendor/bin/pint --dirty             # Only uncommitted files
./vendor/bin/pint --diff=main         # Only modified files from main
./vendor/bin/pint --repair            # Fix and exit non-zero if changes
./vendor/bin/pint app/Models          # Specific directory
./vendor/bin/pint --parallel          # Parallel mode
```

## Configuration

Create `pint.json`:

```json
{
    "preset": "laravel"
}
```

## Presets

- `laravel` (default)
- `per`
- `psr12`
- `symfony`
- `empty`

```bash
./vendor/bin/pint --preset psr12
```

## Rules

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true
        }
    }
}
```

## Excluding Files

```json
{
    "exclude": [
        "my-specific/folder"
    ],
    "notName": [
        "*-my-file.php"
    ],
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

## GitHub Actions

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: shivammathur/setup-php@v2
        with:
          php-version: 8.4
          tools: pint
      - run: pint
      - uses: stefanzweifel/git-auto-commit-action@v6
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/pint)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
