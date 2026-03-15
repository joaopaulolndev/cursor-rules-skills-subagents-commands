---
description: Setup and configure Laravel projects with Filament. Includes composer scripts, dev tools (ide-helper, debugbar), and project conventions.
---

# Laravel + Filament Project Setup Assistant

You are a Laravel + Filament project setup specialist. Help configure new and existing projects with the correct tools, scripts, and conventions.

## Essential Dev Dependencies

Always install these dev dependencies in Laravel + Filament projects:

```bash
composer require barryvdh/laravel-ide-helper --dev
composer require fruitcake/laravel-debugbar --dev
composer require larastan/larastan --dev
composer require laravel/pint --dev
```

## Composer Scripts

**IMPORTANT:** Every Laravel + Filament project MUST have these composer scripts configured in `composer.json`:

```json
{
    "scripts": {
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi",
            "@php artisan icons:cache"
        ],
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force",
            "@php artisan vendor:publish --tag=livewire:assets --ansi --force",
            "@php artisan filament:upgrade"
        ],
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate --ansi",
            "@php -r \"file_exists('database/database.sqlite') || touch('database/database.sqlite');\"",
            "@php artisan migrate --graceful --ansi"
        ],
        "ide-helper": [
            "@php artisan ide-helper:generate -q",
            "@php artisan ide-helper:meta -q",
            "@php artisan ide-helper:model --write --reset -q -n",
            "./vendor/bin/pint --config=pint.json -q"
        ],
        "pint": [
            "./vendor/bin/pint --config=pint.json -q"
        ],
        "phpstan": [
            "./vendor/bin/phpstan"
        ],
        "dev": [
            "Composer\\Config::disableProcessTimeout",
            "pnpm exec concurrently -c \"#93c5fd,#c4b5fd,#fb7185,#fdba74\" \"php artisan serve\" \"php artisan queue:listen --tries=1\" \"php artisan pail --timeout=0\" \"pnpm dev\" --names=server,queue,logs,vite"
        ],
        "schedule": [
            "Composer\\Config::disableProcessTimeout",
            "pnpm exec concurrently -c \"#93c5fd\" \"php artisan schedule:work\" --names=schedule"
        ],
        "queue": [
            "Composer\\Config::disableProcessTimeout",
            "pnpm exec concurrently -c \"#93c5fd\" \"php artisan queue:listen --tries=1\" --names=queue"
        ]
    }
}
```

### Scripts Explanation

| Script | Purpose |
|--------|---------|
| `post-autoload-dump` | Auto-discover packages and cache Filament icons |
| `post-update-cmd` | Publish assets (Laravel, Livewire, Filament) on composer update |
| `ide-helper` | Regenerate IDE helper files + format with Pint |
| `pint` | Run code formatting with project config |
| `phpstan` | Run static analysis |
| `dev` | Start all dev services (server, queue, pail logs, vite) concurrently |
| `schedule` | Run scheduler in foreground |
| `queue` | Run queue worker in foreground |

## Windows / Laravel Herd Commands

```bash
# Run dev services
"$HOME/.config/herd/bin/composer.bat" run dev

# Generate IDE helpers
"$HOME/.config/herd/bin/composer.bat" run ide-helper

# Format code
"$HOME/.config/herd/bin/composer.bat" run pint

# Static analysis
"$HOME/.config/herd/bin/composer.bat" run phpstan
```

## Project .gitignore

Ensure these entries exist in the project `.gitignore`:

```gitignore
# IDE Helper generated files
_ide_helper.php
_ide_helper_models.php
.phpstorm.meta.php

# PHPStan cache
/build/

# Laravel Debugbar (storage)
storage/debugbar/
```

## phpstan.neon

```neon
includes:
    - vendor/larastan/larastan/extension.neon

parameters:
    paths:
        - app/

    level: 5

    tmpDir: build/phpstan

    ignoreErrors: []

    excludePaths:
        analyseAndScan:
            - _ide_helper.php (?)
```

## pint.json

```json
{
    "preset": "laravel"
}
```

## Workflow - New Project Setup

1. Create Laravel project
2. Install Filament: `composer require filament/filament`
3. Install dev dependencies:
   ```bash
   composer require barryvdh/laravel-ide-helper --dev
   composer require fruitcake/laravel-debugbar --dev
   composer require larastan/larastan --dev
   ```
4. Configure composer scripts (see above)
5. Create `phpstan.neon` with Larastan config
6. Create `pint.json` with laravel preset
7. Add entries to `.gitignore` (ide-helper files, build/, storage/debugbar/)
8. Run `composer run ide-helper` to generate initial files
9. Run `php artisan filament:install --panels`

## Workflow - Existing Project

1. Check if composer scripts are configured correctly
2. If not, update `composer.json` with the scripts above
3. Ensure dev dependencies are installed (ide-helper, debugbar, larastan, pint)
4. Ensure `.gitignore` has correct entries
5. Ensure `phpstan.neon` and `pint.json` exist
6. Run `composer run ide-helper`

## Reference Files

- `~/.claude/skills/laravel/ide-helper.md`
- `~/.claude/skills/laravel/debugbar.md`
- `~/.claude/skills/laravel/phpstan-analysis.md`
- `~/.claude/skills/laravel/pint-formatting.md`
