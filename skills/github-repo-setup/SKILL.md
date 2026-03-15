---
description: Update GitHub repo description, topics/tags, and setup .github/FUNDING.yml for sponsors.
---

# GitHub Repository Setup Assistant

You are a GitHub repository setup specialist. Help configure repo metadata (description, topics) and community files (.github/FUNDING.yml).

## Update Repository Description

```bash
# Set description
gh repo edit --description "Your repo description here"

# Set homepage URL
gh repo edit --homepage "https://example.com"

# Set description + homepage together
gh repo edit --description "Description here" --homepage "https://example.com"
```

## Update Repository Topics/Tags

```bash
# Add topics (replaces all existing)
gh repo edit --add-topic "laravel" --add-topic "filament" --add-topic "php"

# Remove a topic
gh repo edit --remove-topic "old-topic"

# View current topics
gh repo view --json repositoryTopics --jq '.repositoryTopics[].name'
```

### Common Topics by Project Type

#### Laravel Project

```bash
gh repo edit --add-topic "laravel" --add-topic "php" --add-topic "filament" --add-topic "livewire"
```

#### Laravel Package

```bash
gh repo edit --add-topic "laravel" --add-topic "php" --add-topic "laravel-package" --add-topic "composer"
```

#### Filament Plugin

```bash
gh repo edit --add-topic "laravel" --add-topic "php" --add-topic "filament" --add-topic "filament-plugin"
```

## GitHub Sponsors (FUNDING.yml)

Create `.github/FUNDING.yml` in the repo root to enable the "Sponsor" button.

### Setup

```bash
# Create .github directory
mkdir -p .github
```

### FUNDING.yml Template

```yaml
# .github/FUNDING.yml
# Uncomment and configure the platforms you use:

# GitHub Sponsors
github: your-username

# Other platforms (optional)
# patreon: your-patreon
# open_collective: your-opencollective
# ko_fi: your-kofi
# tidelift: npm/package-name
# community_bridge: project-name
# liberapay: your-liberapay
# issuehunt: your-username
# polar: your-username
# buy_me_a_coffee: your-username
# custom: ["https://your-custom-link.com"]
```

### Multiple Sponsors

```yaml
# Support multiple GitHub sponsors
github: [user1, user2]

# Mix platforms
github: your-username
ko_fi: your-kofi
custom: ["https://example.com/donate"]
```

## View Current Repo Info

```bash
# Full repo info
gh repo view

# JSON format (specific fields)
gh repo view --json name,description,repositoryTopics,homepageUrl

# Just description
gh repo view --json description --jq '.description'
```

## Visibility Settings

```bash
# Make public
gh repo edit --visibility public

# Make private
gh repo edit --visibility private

# Enable/disable features
gh repo edit --enable-issues --enable-wiki=false --enable-projects=false
```

## Workflow

When asked to setup a GitHub repo:

1. **Check current state**: `gh repo view --json name,description,repositoryTopics,homepageUrl`
2. **Update description**: `gh repo edit --description "..."`
3. **Add topics**: `gh repo edit --add-topic "..."` for each relevant topic
4. **Create FUNDING.yml**: Create `.github/FUNDING.yml` with sponsor config
5. **Commit and push** the FUNDING.yml file

## Reference Files

- `~/.claude/skills/git/github-repo-setup.md`
