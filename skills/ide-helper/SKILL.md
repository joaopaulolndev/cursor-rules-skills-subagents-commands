---
description: Configure and run barryvdh/laravel-ide-helper to generate autocompletion files for Laravel projects.
---

# Laravel IDE Helper Assistant

You are a Laravel IDE Helper specialist. Help configure and run ide-helper to improve autocompletion in IDEs.

## Quick Commands

```bash
# Generate helper file for facades
php artisan ide-helper:generate

# Generate PHPStorm meta file
php artisan ide-helper:meta

# Generate model PHPDocs (write to model files)
php artisan ide-helper:model --write --reset -n

# Generate model PHPDocs (separate file)
php artisan ide-helper:model --nowrite

# Full regeneration (recommended composer script)
composer run ide-helper
```

## Windows / Laravel Herd

```bash
"$HOME/.config/herd/bin/php.bat" artisan ide-helper:generate
"$HOME/.config/herd/bin/php.bat" artisan ide-helper:meta
"$HOME/.config/herd/bin/php.bat" artisan ide-helper:model --write --reset -q -n
"$HOME/.config/herd/bin/composer.bat" run ide-helper
```

## Installation

```bash
composer require barryvdh/laravel-ide-helper --dev
```

## Composer Script (Recommended)

Add to `composer.json`:

```json
{
    "scripts": {
        "ide-helper": [
            "@php artisan ide-helper:generate -q",
            "@php artisan ide-helper:meta -q",
            "@php artisan ide-helper:model --write --reset -q -n",
            "./vendor/bin/pint --config=pint.json -q"
        ]
    }
}
```

This script:
1. Generates `_ide_helper.php` (facades + other classes)
2. Generates `.phpstorm.meta.php` (factory patterns, container resolution)
3. Writes `@property` PHPDocs directly into model files
4. Runs Pint to format the generated/modified code

## .gitignore

Add generated files to `.gitignore`:

```gitignore
_ide_helper.php
_ide_helper_models.php
.phpstorm.meta.php
```

## Commands Detail

### ide-helper:generate

Generates `_ide_helper.php` with PHPDocs for:
- Facades (DB, Cache, Auth, etc.)
- Service container bindings
- Custom macros and mixins

```bash
# Default
php artisan ide-helper:generate

# Custom filename
php artisan ide-helper:generate custom_helper.php

# Include helper helpers (optional)
php artisan ide-helper:generate --helpers
```

### ide-helper:meta

Generates `.phpstorm.meta.php` for:
- `app()` container resolution
- `resolve()` calls
- Factory `::make()` patterns
- Config `::get()` return types

```bash
php artisan ide-helper:meta
```

### ide-helper:model

Generates `@property` and `@method` PHPDocs for Eloquent models:

```bash
# Write PHPDocs directly to model files (recommended)
php artisan ide-helper:model --write --reset -n

# Generate separate _ide_helper_models.php file
php artisan ide-helper:model --nowrite

# Specific models only
php artisan ide-helper:model "App\Models\User" "App\Models\Post"

# With database connection
php artisan ide-helper:model --write --reset -n --db-connection=mysql
```

**Flags:**
- `--write` (-W): Write PHPDocs to model files
- `--nowrite` (-N): Write to `_ide_helper_models.php`
- `--reset` (-R): Replace existing PHPDocs (don't append)
- `-n`: Non-interactive (don't ask for confirmation)
- `-q`: Quiet (suppress output)

### Generated Model PHPDoc Example

```php
/**
 * App\Models\User
 *
 * @property int $id
 * @property string $name
 * @property string $email
 * @property \Illuminate\Support\Carbon|null $email_verified_at
 * @property string $password
 * @property string|null $remember_token
 * @property \Illuminate\Support\Carbon|null $created_at
 * @property \Illuminate\Support\Carbon|null $updated_at
 * @property-read \Illuminate\Database\Eloquent\Collection<int, \App\Models\Post> $posts
 * @property-read int|null $posts_count
 *
 * @method static \Database\Factories\UserFactory factory($count = null, $state = [])
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newModelQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User newQuery()
 * @method static \Illuminate\Database\Eloquent\Builder<static>|User query()
 *
 * @mixin \Eloquent
 */
class User extends Authenticatable
```

## Configuration (Optional)

Publish config:

```bash
php artisan vendor:publish --provider="Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider" --tag=config
```

Key options in `config/ide-helper.php`:

```php
return [
    // Filename for generated helper
    'filename' => '_ide_helper.php',

    // Where to write model PHPDocs
    'model_locations' => ['app/Models'],

    // Include fluent methods on models
    'include_fluent' => true,

    // Write model PHPDocs to model files (instead of separate file)
    'write_model_magic_where' => true,

    // Include factory builder methods
    'include_factory_builders' => true,

    // Extra facades to include
    'extra' => [
        'Debugbar' => 'Fruitcake\LaravelDebugbar\Facades\Debugbar',
    ],

    // Model hooks (custom PHPDoc transformers)
    'model_hooks' => [],
];
```

## When to Run ide-helper

- After `composer install` / `composer update`
- After creating or modifying migrations
- After adding new relationships to models
- After adding new Facades or service bindings
- After running `php artisan migrate`

## Workflow

1. Ensure `barryvdh/laravel-ide-helper` is installed as dev dependency
2. Ensure composer script `ide-helper` is configured
3. Ensure `.gitignore` has `_ide_helper.php`, `_ide_helper_models.php`, `.phpstorm.meta.php`
4. Run `composer run ide-helper` to regenerate all files
5. After modifying models/migrations, run again

Read the detailed skill file at `~/.claude/skills/laravel/ide-helper.md` for advanced configuration.
