<div align="center">

# ⚡ Cursor Config — Laravel & Filament

**Rules · Skills · Agents · Commands ready for Laravel + Filament projects**

[![Laravel](https://img.shields.io/badge/Laravel-FF2D20?style=flat-square&logo=laravel&logoColor=white)](https://laravel.com)
[![Filament](https://img.shields.io/badge/Filament-FDAE4B?style=flat-square&logo=filament&logoColor=black)](https://filamentphp.com)
[![Cursor](https://img.shields.io/badge/Cursor-IDE-000000?style=flat-square&logo=cursor&logoColor=white)](https://cursor.com)
[![Livewire](https://img.shields.io/badge/Livewire-4E56A6?style=flat-square&logo=livewire&logoColor=white)](https://livewire.laravel.com)

</div>

---

## What is this?

A collection of **Cursor IDE** configurations optimized for development with **Laravel**, **Filament**, **Livewire**, and **Pest**. Just extract the `.cursor/` folder into the root of your project and your agent will already be trained on modern PHP ecosystem conventions.

---

## 📁 Structure

```
.cursor/
├── rules/          # Passive guides — activated by context or always-on
│   ├── laravel-filament-project.mdc
│   ├── filament-upgrade.mdc
│   ├── pest-test.mdc
│   └── ... (31 rules)
│
├── skills/         # Active capabilities — invoked by the agent when relevant
│   ├── laravel-filament-project/
│   │   └── SKILL.md
│   ├── pest-test/
│   │   └── SKILL.md
│   └── ... (30 skills)
│
├── agents/         # Autonomous specialists — handle long and complex tasks
│   ├── code-reviewer.md
│   ├── laravel-upgrader.md
│   ├── pest-tester.md
│   └── ... (17 agents)
│
└── commands/       # Reusable workflows triggered with /command-name in chat
    ├── git-commit.md
    ├── laravel-migrate.md
    ├── pest-run.md
    └── ... (11 commands)
```

---

## 🧩 Configuration Types

### Rules — `.cursor/rules/*.mdc`

Passive instructions that Cursor reads automatically. They work like a persistent style guide — no need to invoke anything.

```yaml
---
description: Filament v5 conventions — forms, tables, resources
alwaysApply: false   # activated by context
---
```

| When active | How it works |
|---|---|
| `alwaysApply: true` | Included in every conversation |
| `alwaysApply: false` | Activated when the agent deems it relevant based on `description` |

---

### Skills — `.cursor/skills/<n>/SKILL.md`

Specialized capabilities with detailed step-by-step instructions. The agent invokes them automatically when the task matches the `description`, or you can reference them directly in chat.

```
.cursor/skills/
└── laravel-filament-project/
    └── SKILL.md    ← full content with examples and step-by-step
```

> **Required format:** must be a subfolder containing `SKILL.md` — loose `.mdc` files at the root of `skills/` are not recognized by Cursor.

---

### Agents — `.cursor/agents/*.md`

Autonomous specialists for long and complex tasks. The main agent can delegate to them and run them in parallel.

```yaml
---
name: laravel-upgrader
description: Expert in Laravel version upgrades
model: inherit
---
```

> ⚠️ **Agents are a Nightly channel feature.** To enable: `Cursor Settings → Beta → Update Access → Nightly`

**Available agents:**

| Agent | Specialty |
|---|---|
| `code-reviewer` | PHP/Laravel/Filament code review — security, performance, standards |
| `code-quality` | Code quality analysis, SOLID principles, refactoring |
| `laravel-upgrader` | Laravel version upgrades (10→11→12) |
| `laravel-project-scaffolder` | Full Laravel + Filament project setup |
| `laravel-package-creator` | Laravel package creation from scratch |
| `filament-upgrader` | Filament upgrades v3→v4→v5 |
| `filament-plugin-creator` | Filament plugin creation |
| `pest-tester` | Pest test generation and execution |
| `multi-auth-setup` | Multiple authentication guards configuration |
| `orchestration` | Multi-agent architecture for parallel tasks |
| `seo-technical` | Technical SEO audit and optimization |
| `seo-content` | SEO content strategy and creation |
| `seo-schema` | Structured data and Schema.org |
| `seo-performance` | Core Web Vitals and performance |
| `marketing-analyst` | Data analysis and marketing metrics |
| `marketing-copywriter` | Copywriting and content strategy |
| `marketing-cro` | Conversion rate optimization (CRO) |

---

### Commands — `.cursor/commands/*.md`

Reusable workflows triggered with `/command-name` in chat. Plain Markdown files — **no frontmatter**.

> **Required format:** plain Markdown without `---` at the top. Any frontmatter will break the command.

**Available commands:**

| Command | What it does |
|---|---|
| `/git-commit` | Analyzes staged diff and generates a Conventional Commit message |
| `/git-push` | Safe push with pending commits verification |
| `/git-pr` | Generates a structured Pull Request title and body |
| `/laravel-migrate` | Runs migrations with environment safety check |
| `/laravel-make` | Generates any Laravel file with the correct artisan command |
| `/laravel-tinker` | Prepares and runs code in Tinker |
| `/filament-make-resource` | Generates a complete Filament Resource with auto-improvements |
| `/filament-upgrade` | Guides the upgrade to the next Filament major version |
| `/pest-run` | Runs tests with the right scope for the current context |
| `/pest-generate` | Generates complete Pest tests for the open file |
| `/pest-coverage` | Runs coverage report and identifies gaps |

---

## 🚀 Getting Started

**1. Extract the `.cursor/` folder into the root of your project:**

```bash
# Expected structure after extracting
my-laravel-project/
├── .cursor/
│   ├── rules/
│   ├── skills/
│   ├── agents/
│   └── commands/
├── app/
├── routes/
└── ...
```

**2. Reopen the project in Cursor** — rules and skills are loaded automatically.

**3. For agents**, enable the Nightly channel:
```
Cursor Settings → Beta → Update Access → Nightly
```

**4. To use a command**, type `/` in chat and select the desired command:
```
/git-commit
/pest-run
/laravel-make
```

---

## 📦 Available Skills

<details>
<summary><strong>Laravel</strong></summary>

| Skill | Description |
|---|---|
| `laravel-filament-project` | Full setup: composer scripts, ide-helper, debugbar, phpstan, pint |
| `laravel-multi-auth` | Multiple authentication guards with separate tables |
| `laravel-upgrade` | Step-by-step Laravel version upgrade guide |
| `laravel-package` | Laravel package creation from scratch |
| `laravel-passport` | Laravel Passport integration (OAuth2) |
| `laravel-socialite` | Social login with Laravel Socialite |
| `laravel-horizon` | Laravel Horizon setup for Redis queues |
| `laravel-cashier` | Stripe billing integration |
| `laravel-dusk` | Browser testing with ChromeDriver |
| `laravel-boost` | Laravel Boost for AI-assisted development |
| `laravel-soketi` | WebSockets with Soketi |
| `ide-helper` | Autocompletion generation with barryvdh/laravel-ide-helper |
| `debugbar` | Laravel Debugbar configuration |

</details>

<details>
<summary><strong>Filament</strong></summary>

| Skill | Description |
|---|---|
| `filament-upgrade` | Upgrade v3→v4→v5 with breaking changes |
| `filament-plugin` | Filament plugin creation with branch strategy |
| `filament-multi-auth` | Multiple Filament panels with separate guards |

</details>

<details>
<summary><strong>Livewire</strong></summary>

| Skill | Description |
|---|---|
| `livewire-upgrade` | Livewire upgrade v2→v3→v4 |

</details>

<details>
<summary><strong>Testing & Quality</strong></summary>

| Skill | Description |
|---|---|
| `pest-test` | Pest PHP testing — Laravel, Filament, Livewire |
| `pest-multi-auth` | Testing with multiple authentication guards |
| `phpstan` | Static analysis with Larastan |
| `pint` | Code formatting with Laravel Pint |
| `code-review` | Structured code review with checklist |
| `code-health` | Code health analysis — SOLID, technical debt |

</details>

<details>
<summary><strong>Git & Environment</strong></summary>

| Skill | Description |
|---|---|
| `git-setup` | Git repository initialization |
| `github-repo-setup` | GitHub repository configuration via `gh` CLI |
| `herd-php` | PHP commands with Laravel Herd on Windows |
| `pnpm-build` | Asset building with pnpm |

</details>

<details>
<summary><strong>Other</strong></summary>

| Skill | Description |
|---|---|
| `bugsnag` | Bugsnag integration for any platform |
| `banner-generate` | Banner generation for GitHub repositories |
| `tailwind-upgrade` | Tailwind v3→v4 upgrade |

</details>

---

## 🔧 References

- [Cursor Docs — Rules](https://cursor.com/docs/context/rules)
- [Cursor Docs — Skills](https://cursor.com/docs/context/skills)
- [Cursor Docs — Subagents](https://cursor.com/docs/context/subagents)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Laravel Docs](https://laravel.com/docs)
- [Filament Docs](https://filamentphp.com/docs)

---

<div align="center">

Made with ☕ for the **Laravel + Filament + Cursor** ecosystem

</div>
