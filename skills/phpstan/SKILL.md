---
description: Run PHPStan static analysis on PHP/Laravel projects. Configure rules, baselines, custom extensions, and fix reported issues.
---

# PHPStan Static Analysis Assistant

You are a PHPStan specialist. Help configure, run, and fix static analysis issues in PHP/Laravel projects.

## Quick Commands

```bash
# Run analysis (default level)
vendor/bin/phpstan analyse

# Run with specific level (0-9, 9 = strictest)
vendor/bin/phpstan analyse --level=5

# Analyse specific paths
vendor/bin/phpstan analyse src/ app/

# Generate baseline (ignore existing errors)
vendor/bin/phpstan analyse --generate-baseline

# Run with memory limit
vendor/bin/phpstan analyse --memory-limit=512M

# Clear result cache
vendor/bin/phpstan clear-result-cache

# Run in debug mode
vendor/bin/phpstan analyse --debug

# Output as JSON
vendor/bin/phpstan analyse --error-format=json

# No progress bar (CI)
vendor/bin/phpstan analyse --no-progress
```

## Windows / Laravel Herd

```bash
# Run with Herd PHP
"$HOME/.config/herd/bin/php.bat" vendor/bin/phpstan analyse

# With memory limit
"$HOME/.config/herd/bin/php.bat" -d memory_limit=-1 vendor/bin/phpstan analyse

# Specific PHP version
"$HOME/.config/herd/bin/php84.bat" vendor/bin/phpstan analyse
```

## Installation

### Laravel Project

```bash
composer require --dev larastan/larastan
```

### Generic PHP Project

```bash
composer require --dev phpstan/phpstan
```

### Useful Extensions

```bash
# Laravel (Larastan includes phpstan/phpstan)
composer require --dev larastan/larastan

# Strict rules
composer require --dev phpstan/phpstan-strict-rules

# Deprecation rules
composer require --dev phpstan/phpstan-deprecation-rules

# PHPUnit assertions
composer require --dev phpstan/phpstan-phpunit

# Mockery
composer require --dev phpstan/phpstan-mockery
```

## Configuration (phpstan.neon)

### Laravel Project (with Larastan)

```neon
includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    paths:
        - app/
        - config/
        - database/
        - routes/

    level: 5

    ignoreErrors: []

    excludePaths:
        - app/Console/Kernel.php

    tmpDir: build/phpstan

    checkMissingIterableValueType: false
```

### Generic PHP Project

```neon
parameters:
    paths:
        - src/

    level: 5

    tmpDir: build/phpstan
```

### Filament Plugin

```neon
includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    paths:
        - src/
        - config/
        - database/

    level: 5

    tmpDir: build/phpstan

    ignoreErrors: []
```

## Levels Reference

| Level | Description |
|-------|-------------|
| 0 | Basic checks, unknown classes, unknown functions, unknown methods on `$this`, wrong argument count, always undefined variables |
| 1 | Possibly undefined variables, unknown magic methods and properties on `$this` |
| 2 | Unknown methods on all expressions (not just `$this`), validating PHPDocs |
| 3 | Return types, types assigned to properties |
| 4 | Basic dead code checking - always false `instanceof`, unreachable code after return |
| 5 | Checking types of arguments passed to methods and functions |
| 6 | Report missing typehints |
| 7 | Report partially wrong union types |
| 8 | Report calling methods and accessing properties on nullable types |
| 9 | Be strict about the `mixed` type - no operation on `mixed` allowed |

## Baseline

When introducing PHPStan to an existing project, generate a baseline to ignore current errors and only enforce rules on new code:

```bash
# Generate baseline
vendor/bin/phpstan analyse --generate-baseline

# This creates phpstan-baseline.neon
```

Add to `phpstan.neon`:

```neon
includes:
    - phpstan-baseline.neon
    - vendor/larastan/larastan/extension.neon

parameters:
    level: 5
    paths:
        - app/
```

## Common Fixes

### Undefined Property

```php
// Error: Access to an undefined property App\Models\User::$name
// Fix: Add @property PHPDoc to the model

/**
 * @property string $name
 * @property string $email
 */
class User extends Model {}
```

### Generic Type Missing

```php
// Error: Class Collection missing generic type
// Fix: Add @template or @extends annotation

/** @extends Collection<int, User> */
class UserCollection extends Collection {}
```

### Mixed Type

```php
// Error: Parameter $value has no type
// Fix: Add type declarations
function process(mixed $value): string {
    return (string) $value;
}
```

### Ignore Specific Errors

```neon
parameters:
    ignoreErrors:
        # Ignore specific message
        - '#Call to an undefined method Illuminate\\Database\\Eloquent\\Builder::\w+\(\)#'

        # Ignore in specific file
        -
            message: '#Unsafe usage of new static#'
            path: app/Models/BaseModel.php

        # Ignore by count
        -
            message: '#Access to an undefined property#'
            count: 3
            path: app/Services/LegacyService.php
```

## CI Integration

### GitHub Actions

```yaml
- name: PHPStan
  run: vendor/bin/phpstan analyse --no-progress --error-format=github
```

### Pre-commit (composer script)

```json
{
    "scripts": {
        "analyse": "phpstan analyse --no-progress",
        "test": [
            "@analyse",
            "@php vendor/bin/pest"
        ]
    }
}
```

## Workflow

When asked to run PHPStan or fix analysis errors:

1. Check if `phpstan.neon` exists in the project root
2. If not, create one with sensible defaults for the project type (Laravel, package, etc.)
3. **IMPORTANT:** Ensure `phpstan.neon` has `tmpDir: build/phpstan` configured
4. **IMPORTANT:** Ensure the project's `.gitignore` contains `/build/` to exclude PHPStan cache files. If missing, add it
5. Run the analysis
6. Fix reported errors systematically (highest impact first)
7. If too many pre-existing errors, suggest generating a baseline
8. Re-run to confirm fixes

### .gitignore Check

**Always verify** the project's `.gitignore` includes the PHPStan build directory. Add if missing:

```gitignore
/build/
```

This prevents committing machine-specific PHPStan cache files to the repository. This applies to **all project types**: Laravel apps, packages, and Filament plugins.

Read the detailed skill file at `~/.claude/skills/laravel/phpstan-analysis.md` for comprehensive patterns and Larastan-specific rules.
