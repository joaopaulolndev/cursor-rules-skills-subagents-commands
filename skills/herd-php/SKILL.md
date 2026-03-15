---
description: Run PHP commands with Laravel Herd on Windows. Supports multiple PHP versions (8.1-8.4), Composer, and testing tools.
---

# Laravel Herd PHP Assistant

You are a Laravel Herd specialist for Windows. Help run PHP commands with the correct paths.

## CRITICAL: PHP Version from composer.json — MANDATORY

**Before running ANY PHP command**, ALWAYS check the project's `composer.json` for the minimum PHP version and use the corresponding binary.

### How to determine the correct version

1. Read the `"php"` field in `require` from the project's `composer.json`
2. Use the **minimum version** specified as the PHP binary

### Version mapping

| composer.json `"php"` | Binary to use |
|------------------------|---------------|
| `^8.1` | `php81.bat` |
| `^8.2` | `php82.bat` |
| `^8.2\|^8.3` | `php82.bat` (use minimum) |
| `^8.3` | `php83.bat` |
| `^8.4` | `php84.bat` |

**NEVER use `php.bat` (without version)** — it uses the system default which may not match the project requirement. Always use the versioned binary.

---

## PHP Binary Paths (Git Bash)

```bash
# Specific versions (ALWAYS use the version from composer.json)
"$HOME/.config/herd/bin/php81.bat"  # PHP 8.1
"$HOME/.config/herd/bin/php82.bat"  # PHP 8.2
"$HOME/.config/herd/bin/php83.bat"  # PHP 8.3
"$HOME/.config/herd/bin/php84.bat"  # PHP 8.4

# Composer
"$HOME/.config/herd/bin/composer.bat"
```

## Common Commands

**NOTE:** Replace `phpXX.bat` with the minimum version from the project's `composer.json`.

### Artisan

```bash
# Use the version from composer.json (e.g., "^8.2" → php82.bat)
"$HOME/.config/herd/bin/php82.bat" artisan serve
"$HOME/.config/herd/bin/php82.bat" artisan migrate
"$HOME/.config/herd/bin/php82.bat" artisan make:model User
```

### Composer

```bash
# Install dependencies
"$HOME/.config/herd/bin/composer.bat" install

# Update dependencies
"$HOME/.config/herd/bin/composer.bat" update

# Require package
"$HOME/.config/herd/bin/composer.bat" require laravel/sanctum

# With more memory (for large projects)
"$HOME/.config/herd/bin/php82.bat" -d memory_limit=-1 "$HOME/.config/herd/bin/composer.bat" update

# Using specific PHP version for Composer
"$HOME/.config/herd/bin/php82.bat" "$HOME/.config/herd/bin/composer.bat" install
```

### Testing

```bash
# Pest (use version from composer.json)
"$HOME/.config/herd/bin/php82.bat" vendor/bin/pest

# PHPUnit
"$HOME/.config/herd/bin/php82.bat" vendor/bin/phpunit
```

### Code Quality

```bash
# PHPStan (use version from composer.json)
"$HOME/.config/herd/bin/php82.bat" vendor/bin/phpstan analyse

# Pint (code formatting)
"$HOME/.config/herd/bin/php82.bat" vendor/bin/pint

# Check formatting without changes
"$HOME/.config/herd/bin/php82.bat" vendor/bin/pint --test
```

## Version Compatibility

| Laravel | Minimum PHP | Recommended |
|---------|-------------|-------------|
| 10.x | 8.1 | 8.2+ |
| 11.x | 8.2 | 8.3+ |
| 12.x | 8.2 | 8.3+ |

| Filament | Minimum PHP |
|----------|-------------|
| 3.x | 8.1 |
| 4.x | 8.2 |
| 5.x | 8.2 |

## Check PHP Version

```bash
# Check specific version
"$HOME/.config/herd/bin/php82.bat" -v

# Check project requirement
cat composer.json | grep '"php"'
```

## Per-Project PHP Version

Create `.herd` file in project root:

```json
{
    "php": "8.2"
}
```

## Windows-Specific Notes

### NEVER use heredoc syntax
```bash
# WRONG - creates 'nul' file
git commit -m "$(cat <<'EOF'
message
EOF
)"

# CORRECT - inline message
git commit -m "message"
```

### Always quote paths with spaces
```bash
# WRONG
cd $HOME/My Documents

# CORRECT
cd "$HOME/My Documents"
```

### Avoid cmd.exe for output
```bash
# WRONG - swallows stdout
cmd.exe /c "php artisan --version"

# CORRECT - use full path with versioned binary
"$HOME/.config/herd/bin/php82.bat" artisan --version
```

## Troubleshooting

### Command not found
```bash
# Verify Herd installation
ls "$HOME/.config/herd/bin/"
```

### Memory limit errors
```bash
"$HOME/.config/herd/bin/php82.bat" -d memory_limit=-1 "$HOME/.config/herd/bin/composer.bat" update
```

### Wrong PHP version
```bash
# Always specify version explicitly based on composer.json
"$HOME/.config/herd/bin/php82.bat" artisan serve
```

## Reference Files

- `~/.claude/skills/environment/windows-herd.md`
