---
description: Generate project banner image using banners-cli. Auto-detects project type (CLI, starter kit, Laravel/Filament package) and captures metadata from composer.json.
---

# Banner Generator

You are a banner generation specialist. Use the `banners` CLI (installed globally via Composer) to generate a project banner image saved at `art/{owner}-{repo}.png`.

## CLI Execution — OBRIGATÓRIO

O `banners` CLI é um pacote Laravel Zero instalado globalmente via Composer. No Windows com Git Bash + Laravel Herd, **SEMPRE** usar o caminho completo:

```bash
# Caminho do binário
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" <command> [options]
```

**NUNCA** chamar `banners` diretamente — use sempre o caminho completo com PHP do Herd.

### Verificação de Instalação

Antes de gerar o banner, verificar se o CLI existe:

```bash
# Verificar se o binário existe
ls "$HOME/AppData/Roaming/Composer/vendor/bin/banners" 2>/dev/null
```

Se o binário **NÃO existir**, informar o usuário e sugerir a instalação:

```bash
# Instalar banners-cli globalmente
"$HOME/.config/herd/bin/composer.bat" global require jeffersongoncalves/banners-cli
```

**NÃO prosseguir** com a geração do banner até que o CLI esteja instalado.

---

## Workflow

### 1. Read composer.json to extract project metadata

```bash
# Get project info — use Read tool, NOT cat
```

Extract from `composer.json`:
- **name**: `vendor/package-name` → use `package-name` as banner name (human-readable, capitalize words)
- **description**: use as `--description`
- **owner**: extract from `name` field (before `/`)
- **repo**: extract from `name` field (after `/`)

### 2. Detect project type and determine packageManager

Detect by analyzing `composer.json` and project structure:

#### CLI Tool (Laravel Zero)
- Has `laravel-zero/framework` in require or require-dev
- Has a binary in `bin` field
- **packageManager**: the install command for the CLI

```bash
# If PHAR available via releases
--packageManager="curl -sL https://github.com/{owner}/{repo}/releases/latest/download/{binary}.phar -o {binary}"

# If composer global install
--packageManager="composer global require"
```

#### Starter Kit (filakit, etc.)
- Has `laravel/installer` in require or require-dev
- Project name contains "kit" or has starter kit indicators
- **packageManager**: custom command base (WITHOUT trailing `=`)
- **packageName**: flag + value joined (e.g., `--kit={owner}/{repo}`)

> **IMPORTANT**: The banner service renders `{packageManager} {packageName}` with a space between them.
> If you put `--kit=` at the end of packageManager and `{owner}/{repo}` in packageName, it renders as
> `--kit= owner/repo` with an unwanted space after `=`. To avoid this, put the base command in
> packageManager and the full `--flag=value` in packageName.

```bash
--packageManager="filakit new my-app"
--packageName="--kit={owner}/{repo}"
```

#### Filament Plugin
- Has `filament/filament` or `filament/support` in require
- Has `extra.laravel.providers` with Filament-related providers
- **packageManager**: `composer require`

```bash
--packageManager="composer require"
--packageName="{owner}/{repo}"
```

#### Laravel Package
- Has `illuminate/*` or `laravel/*` in require
- Has `extra.laravel.providers` or `extra.laravel.aliases`
- **packageManager**: `composer require`

```bash
--packageManager="composer require"
--packageName="{owner}/{repo}"
```

#### Default (any other project)
- **packageManager**: `composer require`

```bash
--packageManager="composer require"
--packageName="{owner}/{repo}"
```

### 3. Generate the banner

```bash
# Create art directory
mkdir -p art

# Generate banner
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "{Banner Name}" art/{owner}-{repo}.png \
  --theme=light \
  --pattern=circuitBoard \
  --description="{description from composer.json}" \
  --packageManager="{detected package manager}" \
  --packageName="{owner/repo}" \
  --fileType=png
```

### 4. Update README with banner logo

After generating the banner, add it to the top of `README.md`. If the file already has a banner `<div>` block, replace it. Otherwise, insert before the first line.

The banner URL uses the raw GitHub URL pointing to the `main` branch:

```markdown
<div class="filament-hidden">

![{Banner Name}](https://raw.githubusercontent.com/{owner}/{repo}/main/art/{owner}-{repo}.png)

</div>
```

**Example for `jeffersongoncalves/filakitv5`:**

```markdown
<div class="filament-hidden">

![Filakit v5](https://raw.githubusercontent.com/jeffersongoncalves/filakitv5/main/art/jeffersongoncalves-filakitv5.png)

</div>
```

**Rules:**
- The `<div class="filament-hidden">` wrapper hides the banner inside Filament admin panels (where README is rendered)
- Keep an empty line before and after the `![...]()` image tag
- Place this block at the **very top** of README.md, before the `# Title` heading
- If a banner block already exists (detected by `filament-hidden` div or existing `art/` image), replace it
- Use the `main` branch in the URL (not `master`)

### 5. Verify and show result

```bash
# Check file was created
ls -la art/{owner}-{repo}.png
```

## Banner Name Rules

- Use the **repo name** converted to human-readable format
- Replace hyphens with spaces, capitalize each word
- Examples:
  - `banners-cli` → `Banners CLI`
  - `filakit-cli` → `Filakit CLI`
  - `laravel-settings` → `Laravel Settings`
  - `filakitv5` → `Filakit v5`

## Default Options

Use these defaults unless the project has a `~/.banners-cli/config.json`:

| Option | Default |
|--------|---------|
| `--theme` | `light` |
| `--pattern` | `circuitBoard` |
| `--fontSize` | `96px` |
| `--fileType` | `png` |

The user can override any option via arguments passed to the skill.

## Examples

### CLI Tool

```bash
# For banners-cli (Laravel Zero CLI)
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "Banners CLI" art/jeffersongoncalves-banners-cli.png \
  --theme=light \
  --pattern=circuitBoard \
  --description="CLI tool to generate banner images using beyondcode/banners service." \
  --packageManager="composer global require" \
  --packageName="jeffersongoncalves/banners-cli" \
  --fileType=png
```

### Starter Kit

```bash
# For filakitv5 (starter kit)
# NOTE: --packageManager has the base command, --packageName has --kit=value
# This avoids an unwanted space: "filakit new my-app --kit=jeffersongoncalves/filakitv5"
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "Filakit v5" art/jeffersongoncalves-filakitv5.png \
  --theme=light \
  --pattern=circuitBoard \
  --description="Laravel + Filament v5 Starter Kit" \
  --packageManager="filakit new my-app" \
  --packageName="--kit=jeffersongoncalves/filakitv5" \
  --fileType=png
```

### Laravel/Filament Package

```bash
# For a Filament plugin
"$HOME/.config/herd/bin/php.bat" "$HOME/AppData/Roaming/Composer/vendor/bin/banners" banner:generate "My Plugin" art/vendor-my-plugin.png \
  --theme=light \
  --pattern=circuitBoard \
  --description="A Filament plugin for something." \
  --packageManager="composer require" \
  --packageName="vendor/my-plugin" \
  --fileType=png
```

## Arguments

Any extra arguments passed when invoking the skill are forwarded as overrides:

```
/banner-generate --theme=dark --pattern=texture
```

These override the auto-detected defaults.
