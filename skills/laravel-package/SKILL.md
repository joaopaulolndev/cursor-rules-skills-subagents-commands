---
description: Create Laravel packages/plugins with proper structure, Service Provider, Facade, and testing setup.
---

# Laravel Package Creator

You are a Laravel package development specialist. Help create new packages with best practices.

## Important: Branch Strategy

Laravel packages use a **single `main` branch** because they can support multiple Laravel versions via composer:

```json
"illuminate/support": "^10.0|^11.0|^12.0"
```

## Creating New Package

### 1. Initialize Project Structure

```bash
# Create package directory
mkdir my-laravel-package && cd my-laravel-package

# Initialize git
git init

# Create directory structure
mkdir -p src/{Commands,Facades,Http/Controllers,Http/Middleware,Models}
mkdir -p config database/migrations
mkdir -p resources/{lang/en,lang/pt_BR,views}  # lang SEMPRE dentro de resources/
mkdir -p routes
mkdir -p tests/{Feature,Unit}
```

### 2. Create composer.json

```json
{
    "name": "vendor/my-package",
    "description": "Laravel package for [functionality]",
    "keywords": ["laravel", "package"],
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your@email.com"
        }
    ],
    "require": {
        "php": "^8.2",
        "illuminate/support": "^11.0|^12.0",
        "spatie/laravel-package-tools": "^1.14.0"
    },
    "require-dev": {
        "larastan/larastan": "^3.0",
        "laravel/pint": "^1.21",
        "orchestra/testbench": "^10.0|^11.0",
        "pestphp/pest": "^3.0",
        "pestphp/pest-plugin-laravel": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "Vendor\\MyPackage\\": "src"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Vendor\\MyPackage\\Tests\\": "tests"
        }
    },
    "scripts": {
        "analyse": "vendor/bin/phpstan analyse",
        "format": "vendor/bin/pint",
        "test": "vendor/bin/pest"
    },
    "extra": {
        "laravel": {
            "providers": [
                "Vendor\\MyPackage\\MyPackageServiceProvider"
            ],
            "aliases": {
                "MyPackage": "Vendor\\MyPackage\\Facades\\MyPackage"
            }
        }
    }
}
```

### 3. Create Service Provider (with Spatie Package Tools)

```php
<?php

namespace Vendor\MyPackage;

use Spatie\LaravelPackageTools\Package;
use Spatie\LaravelPackageTools\PackageServiceProvider;
use Vendor\MyPackage\Commands\MyCommand;

class MyPackageServiceProvider extends PackageServiceProvider
{
    public function configurePackage(Package $package): void
    {
        $package
            ->name('my-package')
            ->hasConfigFile()
            ->hasViews()
            ->hasTranslations()
            ->hasMigration('create_my_table')
            ->hasCommand(MyCommand::class)
            ->hasRoute('web')
            ->hasRoute('api');
    }

    public function packageRegistered(): void
    {
        $this->app->singleton('my-package', function () {
            return new MyPackage();
        });
    }
}
```

### 4. Create Facade

```php
<?php

namespace Vendor\MyPackage\Facades;

use Illuminate\Support\Facades\Facade;

/**
 * @see \Vendor\MyPackage\MyPackage
 */
class MyPackage extends Facade
{
    protected static function getFacadeAccessor(): string
    {
        return 'my-package';
    }
}
```

### 5. Create Main Class

```php
<?php

namespace Vendor\MyPackage;

class MyPackage
{
    public function doSomething(string $input): string
    {
        return "Processed: {$input}";
    }
}
```

### 6. Create Config File

```php
<?php
// config/my-package.php

return [
    'option_one' => env('MY_PACKAGE_OPTION_ONE', 'default'),
    'option_two' => env('MY_PACKAGE_OPTION_TWO', true),
];
```

### 7. Setup Tests

**tests/Pest.php:**
```php
<?php

use Vendor\MyPackage\Tests\TestCase;

uses(TestCase::class)->in('Feature', 'Unit');
```

**tests/TestCase.php:**
```php
<?php

namespace Vendor\MyPackage\Tests;

use Orchestra\Testbench\TestCase as Orchestra;
use Vendor\MyPackage\MyPackageServiceProvider;

abstract class TestCase extends Orchestra
{
    protected function getPackageProviders($app): array
    {
        return [MyPackageServiceProvider::class];
    }

    protected function getEnvironmentSetUp($app): void
    {
        config()->set('database.default', 'testing');
        config()->set('database.connections.testing', [
            'driver' => 'sqlite',
            'database' => ':memory:',
        ]);
    }
}
```

### 8. Generate Banner Image

After creating the package, generate a banner image and add it to the README:

```bash
# Create art directory
mkdir -p art

# Generate banner (extract name/description from composer.json)
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "{Package Name}" art/{owner}-{repo}.png \
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

![{Package Name}](https://raw.githubusercontent.com/{owner}/{repo}/main/art/{owner}-{repo}.png)

</div>

# Package Name
```

**Rules:**
- Banner name: repo name converted to Title Case (e.g., `laravel-cep` → `Laravel Cep`)
- URL branch: always `main` (Laravel packages use single `main` branch)
- The `<div class="filament-hidden">` wrapper hides the banner inside Filament admin panels
- If the package is a dev dependency (e.g., Faker providers, testing tools), use `--packageManager="composer require --dev"`

## Publishing to Packagist

1. Create GitHub repository
2. Register at https://packagist.org
3. Submit package with GitHub URL
4. Configure webhook for auto-updates

## Commands

```bash
# Install dependencies
composer install

# Run tests
composer test

# Format code
composer format

# Static analysis
composer analyse
```

## Reference Files

- `~/.claude/skills/laravel/plugin-creation.md`
