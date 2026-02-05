# Contributing

## Report an Error or Bug

To report issues, navigate to the [issue page](https://github.com/tempestphp/tempest-framework/issues) and provide:

- Detailed context about the problem
- Your environment details
- Tempest version and affected component
- Optionally, a PR with a failing test for the "Perfect Storm" label

The team will label, assign, and respond promptly. Issues receive closure after 30 days of inactivity.

## Request a Feature

Submit feature requests via the [issue page](https://github.com/tempestphp/tempest-framework/issues) with:

- Detailed feature description
- Use cases and benefits

Accepted features receive an "Uncharted waters" label, signaling community contribution opportunities.

## Contribute Documentation

Documentation contributions of any size are welcomed, from typos to substantial additions.

**Process:**
1. Set up locally and navigate to `/docs`
2. Edit documentation consistently with existing style
3. Spell-check your work
4. Optionally symlink via `tempest docs:symlink` and preview with `bun run dev`
5. Open a pull request

The team reviews via GitHub, requests revisions if needed, and merges accepted contributions into the next release.

## Contribute Code

Ensure contributions address approved features or confirmed bugs to align with Tempest's vision.

**Requirements:**
- Set up Tempest locally
- Write tests verifying functionality
- Run `composer qa` for style compliance
- Create a pull request, referencing issues with "Fixes #xxx"

**Pull request titles** must follow conventional commits for readable changelog generation.

### Setting Up Locally

**Prerequisites:**
- PHP
- Composer
- Bun or Node
- Fork and clone the repository

**Commands:**
```bash
cd /path/to/your/clone
composer update
bun install
bun dev
```

#### Symlink Local Tempest to Another Application

Modify `composer.json`:
```json
{
  "repositories": [
    {
      "type": "path",
      "url": "/path/to/your/clone"
    }
  ],
  "minimum-stability": "dev",
  "prefer-stable": true
}
```

Then run: `composer require "tempest/framework:*"`

For JavaScript packages: `bun install /path/to/your/clone/package`

Run `bun dev` in the framework root to reflect changes without rebuilding.

## Code Style and Conventions

Tempest uses a modified PSR-12 with automated enforcement via Mago and Rector.

### Final and Readonly as Default

Classes should be `final` and `readonly` whenever possible, promoting immutability and preventing unintended logic changes.

### Acronym Casing

Following modified .NET best practices:

- Two-to-three character acronyms: capitalize all except first word in camel case (`IPAddress` vs. `ipAddress`)
- Four+ character acronyms: capitalize only first character (`UuidGenerator` vs. `uuidGenerator`)
- Camel-cased identifiers: no capitalization (`Uuid`, `dbUsername`)

### Validation Classes

Error messages should **not include ending punctuation** (periods, exclamation marks, question marks).

### Exception Classes

Think of exceptions as events using past-tense subject-verb naming (`DatabaseOperationFailed`, `AuthenticatedUserWasMissing`).

**Guidelines:**
- No `Exception` suffix in class names
- Extend PHP's `\Exception`
- Define marker interfaces when appropriate (`CacheException`, `DatabaseException`)
- Set messages within the exception class
- Accept only context-specific constructor arguments

**Example:**
```php
final class LockAcquisitionTimedOut extends Exception implements CacheException
{
    public function __construct(
        public readonly string $key,
    ) {
        parent::__construct("Lock with key `{$key}` could not be acquired on time.");
    }
}
```

## Release Workflow

Tempest uses sub-splits enabling individual package installation.

### Workflow Steps

1. **Trigger:** PR merge or tag creation runs `.github/workflows/subsplit-packages.yml`
2. **Retrieval:** `bin/get-packages` returns JSON with directory, name, package, organization, and repository
3. **Matrix:** Results create an action matrix for every package
4. **Split:** `symplify/monorepo-split-github-action@v2.3.0` pushes changes to sub-split repositories with appropriate tagging

## Commit and Merge Conventions

All commits must follow the [conventional commit specification](https://www.conventionalcommits.org/en/) for automated changelog generation.

### Commit Descriptions

Descriptions should not start with capitals and use imperative mood:

### Commit Scopes

Common scopes include:
- `feat` — new feature
- `fix` — bug fix
- `refactor` — code changes (non-feature, non-bugfix)
- `docs` — documentation changes
- `perf` — performance improvements
- `test` — testing code
- `style` — code style refactoring (not CSS)
- `ci` — CI pipeline changes
- `chore` — everything else

**Examples:**
```
feat(support): add `StringHelper` class
feat(support/string): add `uuid` method
perf(discovery): improve cache efficiency
refactor(highlight): improve code readability
docs: mention new `highlight` package
chore: update dependencies
style: apply php-cs-fixer
```

### Pull Requests

Pull request titles should be explicit to ease reviews. While conventional commits aren't required for PRs, they streamline review. Core contributors rename PRs to conventional commits before squash-merging to maintain clean history.
