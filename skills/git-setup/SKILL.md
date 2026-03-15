---
description: Setup Git repositories with proper branch conventions for Laravel projects, Laravel packages, and Filament plugins.
---

# Git Repository Setup Assistant

You are a Git setup specialist. Help configure repositories with proper conventions.

## Branch Convention Summary

| Project Type | Default Branch | Multiple Branches |
|--------------|----------------|-------------------|
| Laravel Project | `main` | No |
| Laravel Package | `main` | No |
| Filament Plugin | `1.x` | Yes (`1.x`, `2.x`, `3.x`) |

## Setup: Laravel Project

```bash
# Create project (already has git initialized)
composer create-project laravel/laravel my-project
cd my-project

# First commit (if needed)
git add .
git commit -m "Initial commit"

# Connect to GitHub
git remote add origin git@github.com:username/my-project.git
git push -u origin main
```

## Setup: Laravel Package

```bash
# Create directory
mkdir my-package && cd my-package

# Initialize git with main branch
git init

# Create structure
mkdir -p src config tests

# Create initial files
touch composer.json README.md LICENSE .gitignore

# First commit
git add .
git commit -m "Initial commit - Laravel package structure"

# Connect to GitHub
git remote add origin git@github.com:username/my-package.git
git push -u origin main
```

## Setup: Filament Plugin

```bash
# Create directory
mkdir my-filament-plugin && cd my-filament-plugin

# Initialize git with 1.x branch (NOT main!)
git init
git checkout -b 1.x

# Create structure
mkdir -p src/{Actions,Forms/Components,Tables/Columns,Pages,Resources,Widgets}
mkdir -p config resources/{css,js,lang,views} tests

# Create initial files
touch composer.json README.md LICENSE .gitignore

# First commit
git add .
git commit -m "Initial commit - Filament plugin structure"

# Connect to GitHub
git remote add origin git@github.com:username/my-filament-plugin.git
git push -u origin 1.x

# IMPORTANT: Set 1.x as default branch on GitHub
# Settings → Branches → Default branch → Change to 1.x
```

## Creating New Version Branch (Filament Plugin)

```bash
# Create branch for Filament 4 support
git checkout 1.x
git checkout -b 2.x
# Update composer.json: "filament/filament": "^3.0" → "^4.0"
git add .
git commit -m "feat: upgrade to Filament v4 compatibility"
git push -u origin 2.x

# Create branch for Filament 5 support
git checkout 2.x
git checkout -b 3.x
# Update composer.json: "filament/filament": "^4.0" → "^5.0"
git add .
git commit -m "feat: upgrade to Filament v5 compatibility"
git push -u origin 3.x
```

## Commit Message Convention

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation |
| `style` | Formatting (no code change) |
| `refactor` | Code refactoring |
| `test` | Tests |
| `chore` | Maintenance |
| `build` | Build system or dependencies |

### Examples

```bash
# New feature
git commit -m "feat: add WhatsApp authentication provider"

# Bug fix
git commit -m "fix: resolve code expiry validation issue"

# Dependency update
git commit -m "build(deps): upgrade filament from ^4.0 to ^5.0"

# With co-author
git commit -m "feat: add MFA support

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

## .gitignore Templates

### Laravel Project
```gitignore
/vendor/
/node_modules/
/public/build/
/public/hot
/storage/*.key
.env
.env.backup
.phpunit.result.cache
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
.idea/
.vscode/
```

### Laravel/Filament Package
```gitignore
/vendor/
/node_modules/
/.phpunit.cache/
/build/
composer.lock
.phpunit.result.cache
.DS_Store
.idea/
.vscode/
.env
```

## GitHub Actions: Multiple Branches

For Filament plugins with version branches:

```yaml
# .github/workflows/tests.yml
name: Tests

on:
  push:
    branches: [1.x, 2.x, 3.x]
  pull_request:
    branches: [1.x, 2.x, 3.x]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php: [8.2, 8.3]
    steps:
      - uses: actions/checkout@v4
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
      - run: composer install
      - run: vendor/bin/pest
```

## Tagging Releases

```bash
# For branch 1.x (Filament 3)
git checkout 1.x
git tag 1.0.0
git push origin 1.0.0

# For branch 2.x (Filament 4)
git checkout 2.x
git tag 2.0.0
git push origin 2.0.0

# For branch 3.x (Filament 5)
git checkout 3.x
git tag 3.0.0
git push origin 3.0.0
```

## Reference Files

- `~/.claude/skills/git/best-practices.md`
