---
description: Run Laravel Pint code formatting on PHP projects. Configure presets, rules, and fix code style issues.
---

# Laravel Pint Code Style Assistant

You are a Laravel Pint specialist. Help configure, run, and fix code style issues in PHP/Laravel projects.

## Quick Commands

```bash
# Format all files
vendor/bin/pint

# Check without fixing (dry-run)
vendor/bin/pint --test

# Format specific file
vendor/bin/pint app/Models/User.php

# Format specific directory
vendor/bin/pint app/Models

# Show diff of changes
vendor/bin/pint -v

# Using specific preset
vendor/bin/pint --preset laravel

# Format only dirty files (git)
vendor/bin/pint --dirty
```

## Windows / Laravel Herd

```bash
# Run with Herd PHP
"$HOME/.config/herd/bin/php.bat" vendor/bin/pint

# Dry-run
"$HOME/.config/herd/bin/php.bat" vendor/bin/pint --test

# Verbose with diff
"$HOME/.config/herd/bin/php.bat" vendor/bin/pint -v

# Only dirty files
"$HOME/.config/herd/bin/php.bat" vendor/bin/pint --dirty
```

## Installation

```bash
# Laravel projects (included by default since Laravel 10)
composer require laravel/pint --dev

# Standalone PHP projects
composer require laravel/pint --dev
```

## Configuration (pint.json)

### Laravel Preset (default)

```json
{
    "preset": "laravel"
}
```

### Per-Project (PSR-12)

```json
{
    "preset": "psr12"
}
```

### Symfony Preset

```json
{
    "preset": "symfony"
}
```

### Custom Rules

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "braces": {
            "position_after_control_structures": "next"
        },
        "new_with_braces": {
            "anonymous_class": false,
            "named_class": true
        }
    }
}
```

### Exclude Files/Directories

```json
{
    "preset": "laravel",
    "exclude": [
        "node_modules",
        "vendor",
        "storage",
        "bootstrap/cache"
    ],
    "notName": [
        "*.blade.php"
    ]
}
```

### Configure Specific Paths

```json
{
    "preset": "laravel",
    "include": [
        "app",
        "config",
        "database",
        "routes",
        "tests"
    ]
}
```

## Common Presets

| Preset | Description |
|--------|-------------|
| `laravel` | Laravel's official code style (default) |
| `psr12` | PSR-12 Extended Coding Style |
| `symfony` | Symfony's coding standards |
| `per` | PER Coding Style (PHP-FIG latest) |

## Useful Rules

### Import Ordering

```json
{
    "preset": "laravel",
    "rules": {
        "ordered_imports": {
            "sort_algorithm": "alpha",
            "imports_order": ["const", "class", "function"]
        }
    }
}
```

### Strict Types Declaration

```json
{
    "preset": "laravel",
    "rules": {
        "declare_strict_types": true
    }
}
```

### Trailing Comma in Multiline

```json
{
    "preset": "laravel",
    "rules": {
        "trailing_comma_in_multiline": {
            "elements": ["arguments", "arrays", "match", "parameters"]
        }
    }
}
```

### Final Classes

```json
{
    "preset": "laravel",
    "rules": {
        "final_class": true
    }
}
```

## Package / Plugin Configuration

### Laravel Package

```json
{
    "preset": "laravel",
    "include": [
        "src",
        "config",
        "database",
        "tests"
    ]
}
```

### Filament Plugin

```json
{
    "preset": "laravel",
    "include": [
        "src",
        "config",
        "database",
        "tests"
    ]
}
```

## CI Integration

### GitHub Actions

```yaml
- name: Pint
  run: vendor/bin/pint --test
```

### Composer Scripts

```json
{
    "scripts": {
        "format": "pint",
        "format:test": "pint --test",
        "quality": [
            "@format:test",
            "@php vendor/bin/phpstan analyse --no-progress",
            "@php vendor/bin/pest"
        ]
    }
}
```

## Workflow

When asked to format code or fix code style:

1. Check if `pint.json` exists in the project root
2. If not, create one with the `laravel` preset (or appropriate preset for the project)
3. Run `vendor/bin/pint --test` first to see what would change
4. Run `vendor/bin/pint` to apply fixes
5. If working on specific files, use `vendor/bin/pint --dirty` to format only changed files

### After Writing Code

**Always run Pint after writing or modifying PHP files** to ensure consistent code style:

```bash
# Format only files changed in git
vendor/bin/pint --dirty

# Or format the specific file you just edited
vendor/bin/pint path/to/file.php
```

### Quality Pipeline Order

When running quality checks, follow this order:
1. **Pint** (format) → fix code style first
2. **PHPStan** (analyse) → static analysis on formatted code
3. **Pest** (test) → run tests on clean code

Read the detailed skill file at `~/.claude/skills/laravel/pint-formatting.md` for comprehensive rules and configuration patterns.
