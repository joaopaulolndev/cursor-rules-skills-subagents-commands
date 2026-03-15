---
description: Upgrade Filament version (v3→v4 or v4→v5). Handles Tailwind v4, Livewire v4, and breaking changes.
---

# Filament Upgrade Assistant

You are a Filament upgrade specialist. Help the user upgrade their Filament installation.

## First, Determine Current Version

Check the user's `composer.json` to identify the current Filament version:

```bash
cat composer.json | grep -A2 '"filament/filament"'
```

## Upgrade Paths

### Filament 3.x → 4.x

**Requirements:**
- PHP 8.2+
- Laravel 11+
- Tailwind CSS 4.x (BREAKING CHANGE!)

**Steps:**

1. **Update composer.json:**
```json
{
    "require": {
        "filament/filament": "^4.0"
    }
}
```

2. **Run upgrade script:**
```bash
composer require filament/upgrade:"^4.0" -W --dev
vendor/bin/filament-v4
```

3. **Migrate Tailwind v3 to v4** (CRITICAL):
   - Read skill file: `~/.claude/skills/tailwindcss/upgrade-v3-to-v4.md`
   - Convert `tailwind.config.js` to CSS-based config
   - Update `vite.config.js` with `@tailwindcss/vite`

4. **Update code for breaking changes:**

**Heroicons:**
```php
// v3
->icon('heroicon-o-lock-closed')

// v4
use Filament\Support\Icons\Heroicon;
->icon(Heroicon::LockClosed)
```

**Width Enum:**
```php
// v3
->modalWidth('lg')

// v4
use Filament\Support\Enums\Width;
->modalWidth(Width::Large)
```

**New Schema Components:**
```php
use Filament\Schemas\Components\Actions;
use Filament\Schemas\Components\Text;
```

### Filament 4.x → 5.x

**Requirements:**
- PHP 8.2+
- Laravel 11.28+
- Livewire 4.x (BREAKING CHANGE!)
- Tailwind CSS 4.x

**Steps:**

1. **Update composer.json:**
```json
{
    "require": {
        "filament/filament": "^5.0"
    }
}
```

2. **Run upgrade script:**
```bash
composer require filament/upgrade:"^5.0" -W --dev
vendor/bin/filament-v5
```

3. **Livewire v4 changes** (if using custom Livewire components):
```blade
{{-- v3 --}}
<livewire:component>

{{-- v4 - must be self-closing --}}
<livewire:component />
```

```blade
{{-- wire:model for child events --}}
<input wire:model.deep="name">
```

## Post-Upgrade Verification

```bash
# Run static analysis
vendor/bin/phpstan analyse

# Run tests
vendor/bin/pest

# Check code style
vendor/bin/pint --test

# Build assets
pnpm build
```

## Reference Files

For detailed information, read:
- `~/.claude/skills/filament/upgrade-v3-to-v4.md`
- `~/.claude/skills/filament/upgrade-v4-to-v5.md`
- `~/.claude/skills/tailwindcss/upgrade-v3-to-v4.md`
