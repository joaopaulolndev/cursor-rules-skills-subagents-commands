---
description: Upgrade Laravel versions (5.x through 13.x). Handles breaking changes, new features, and migration steps.
---

# Laravel Upgrade Assistant

You are a Laravel upgrade specialist. Help the user upgrade their Laravel installation.

## First, Determine Current Version

```bash
php artisan --version
cat composer.json | grep -A2 '"laravel/framework"'
```

## Version Requirements Summary

| Laravel | PHP | Release |
|---------|-----|---------|
| 9.x | 8.0+ | Feb 2022 |
| 10.x | 8.1+ | Feb 2023 |
| 11.x | 8.2+ | Mar 2024 |
| 12.x | 8.2+ | Feb 2025 |
| 13.x | 8.3+ | Feb 2026 |

## Common Upgrade Paths

### Laravel 10.x → 11.x

**Major Changes:**
- Streamlined application structure
- Removed `app/Http/Kernel.php`
- Removed `app/Console/Kernel.php`
- Single `AppServiceProvider` by default
- New `bootstrap/app.php` configuration

**Steps:**

1. **Update composer.json:**
```json
{
    "require": {
        "php": "^8.2",
        "laravel/framework": "^11.0"
    }
}
```

2. **Update bootstrap/app.php:**
```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Configure middleware
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Configure exceptions
    })
    ->create();
```

3. **Update Model casts (recommended):**
```php
// Before
protected $casts = ['options' => 'array'];

// After (Laravel 11+)
protected function casts(): array
{
    return ['options' => 'array'];
}
```

### Laravel 11.x → 12.x

**Major Changes:**
- React 19 / Vue 3.5 support
- Laravel Reverb (WebSockets)
- Improved Concurrency facade

**Steps:**

1. **Update composer.json:**
```json
{
    "require": {
        "laravel/framework": "^12.0"
    }
}
```

2. **New Concurrency feature:**
```php
use Illuminate\Support\Facades\Concurrency;

[$users, $posts] = Concurrency::run([
    fn () => User::all(),
    fn () => Post::all(),
]);
```

## Upgrade Commands

```bash
# Update composer.json version
composer require laravel/framework:^12.0 -W

# Clear all caches
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Run migrations
php artisan migrate

# Regenerate autoload
composer dump-autoload
```

## Post-Upgrade Verification

```bash
# Check version
php artisan --version

# Run tests
php artisan test

# Check for deprecations
php artisan about
```

## Recommended Tools

- **Laravel Shift**: https://laravelshift.com (automated upgrades)
- **Rector PHP**: `composer require rector/rector --dev`

## Reference Files

- `~/.claude/skills/laravel/upgrade-guide.md` (complete guide 5.x → 13.x)
