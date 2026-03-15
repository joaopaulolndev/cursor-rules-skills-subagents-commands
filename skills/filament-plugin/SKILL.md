---
description: Create or upgrade Filament plugins with proper branch strategy (1.x, 2.x, 3.x for different Filament versions).
---

# Filament Plugin Creator & Manager

You are a Filament plugin development specialist. Help create new plugins or manage existing ones.

## Important: Branch Strategy

Filament plugins use version branches due to breaking changes:

| Branch | Filament | Tailwind | Livewire |
|--------|----------|----------|----------|
| `1.x` | ^3.0 | 3.x | 3.x |
| `2.x` | ^4.0 | **4.x** | 3.x |
| `3.x` | ^5.0 | **4.x** | **4.x** |

## Creating New Plugin

### 1. Initialize Project Structure

```bash
# Create plugin directory
mkdir my-filament-plugin && cd my-filament-plugin

# Initialize git with 1.x branch (Filament 3)
git init
git checkout -b 1.x

# Create directory structure
mkdir -p src/{Actions,Commands,Components,Concerns,Contracts,Facades}
mkdir -p src/{Forms/Components,Tables/Columns,Pages,Resources,Widgets}
mkdir -p config database/migrations
mkdir -p resources/{css,js,lang/en,lang/pt_BR,views}
mkdir -p tests/{Feature,Fixtures}
```

### 2. Create composer.json

```json
{
    "name": "vendor/my-filament-plugin",
    "description": "Filament plugin for [functionality]",
    "keywords": ["laravel", "filament", "filament-plugin"],
    "license": "MIT",
    "require": {
        "php": "^8.2",
        "filament/filament": "^3.0",
        "spatie/laravel-package-tools": "^1.14.0"
    },
    "require-dev": {
        "larastan/larastan": "^3.0",
        "laravel/pint": "^1.21",
        "orchestra/testbench": "^8.0|^9.0",
        "pestphp/pest": "^3.0",
        "pestphp/pest-plugin-laravel": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "Vendor\\MyFilamentPlugin\\": "src"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Vendor\\MyFilamentPlugin\\Tests\\": "tests"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "Vendor\\MyFilamentPlugin\\MyFilamentPluginServiceProvider"
            ]
        }
    }
}
```

### 3. Create Service Provider

```php
<?php

namespace Vendor\MyFilamentPlugin;

use Spatie\LaravelPackageTools\Package;
use Spatie\LaravelPackageTools\PackageServiceProvider;

class MyFilamentPluginServiceProvider extends PackageServiceProvider
{
    public static string $name = 'my-filament-plugin';

    public function configurePackage(Package $package): void
    {
        $package
            ->name(static::$name)
            ->hasConfigFile()
            ->hasViews()
            ->hasTranslations();
    }
}
```

### 4. Create Plugin Class

```php
<?php

namespace Vendor\MyFilamentPlugin;

use Filament\Contracts\Plugin;
use Filament\Panel;

class MyFilamentPlugin implements Plugin
{
    public function getId(): string
    {
        return 'my-filament-plugin';
    }

    public function register(Panel $panel): void
    {
        $panel
            ->pages([])
            ->resources([])
            ->widgets([]);
    }

    public function boot(Panel $panel): void {}

    public static function make(): static
    {
        return app(static::class);
    }
}
```

### 5. Generate Banner Image

After creating the plugin, generate a banner image and add it to the README:

```bash
# Create art directory
mkdir -p art

# Generate banner (extract name/description from composer.json)
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "{Plugin Name}" art/{owner}-{repo}.png \
  --theme=light \
  --pattern=circuitBoard \
  --description="{description from composer.json}" \
  --packageManager="composer require" \
  --packageName="{owner}/{repo}" \
  --fileType=png
```

Then add the banner to the **top** of `README.md`, before the `# Title` heading:

```markdown
<div class="filament-hidden">

![{Plugin Name}](https://raw.githubusercontent.com/{owner}/{repo}/{branch}/art/{owner}-{repo}.png)

</div>

# Plugin Name
```

**Rules:**
- Banner name: repo name converted to Title Case (e.g., `filament-settings` → `Filament Settings`)
- URL branch: use the **current branch** (`1.x`, `2.x`, or `3.x`) — Filament plugins have version branches
- The `<div class="filament-hidden">` wrapper hides the banner inside Filament admin panels
- Generate the banner once on the first branch, then copy `art/` when creating new version branches

## Upgrading Plugin to New Filament Version

### From 1.x (Filament 3) to 2.x (Filament 4)

```bash
# Create new branch
git checkout 1.x
git checkout -b 2.x

# Update composer.json
# Change: "filament/filament": "^3.0" → "^4.0"
# Change: "orchestra/testbench": "^8.0|^9.0" → "^9.0|^10.0"

# Update code for breaking changes (Heroicons, Width enum, etc.)

# Commit and push
git add .
git commit -m "feat: upgrade to Filament v4 compatibility"
git push -u origin 2.x
```

### From 2.x (Filament 4) to 3.x (Filament 5)

```bash
git checkout 2.x
git checkout -b 3.x

# Update: "filament/filament": "^4.0" → "^5.0"
# Update: "orchestra/testbench": "^9.0|^10.0" → "^10.0|^11.0"

git commit -m "feat: upgrade to Filament v5 compatibility"
git push -u origin 3.x
```

## GitHub Configuration

1. Set default branch to `1.x` (or latest version branch)
2. Create releases with tags: `1.0.0`, `2.0.0`, `3.0.0`

## Reference Files

- `~/.claude/skills/filament/plugin-creation.md`
- `~/.claude/skills/filament/plugin-upgrade-branches.md`
