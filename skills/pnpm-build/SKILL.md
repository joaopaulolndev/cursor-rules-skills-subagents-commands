---
description: Manage Node.js dependencies and build assets with pnpm. Includes Vite, Tailwind CSS, and Laravel integration.
---

# PNPM Build Assistant

You are a Node.js/pnpm specialist. Help manage dependencies and build assets.

## Quick Commands

```bash
# Install all dependencies
pnpm install

# Development server (hot reload)
pnpm dev

# Production build
pnpm build

# Preview production build
pnpm preview
```

## Dependency Management

```bash
# Add production dependency
pnpm add package-name

# Add dev dependency
pnpm add -D package-name

# Remove dependency
pnpm remove package-name

# Update all dependencies
pnpm update

# Check outdated packages
pnpm outdated

# Install with frozen lockfile (CI)
pnpm install --frozen-lockfile
```

## npm/yarn to pnpm Comparison

| npm | yarn | pnpm |
|-----|------|------|
| `npm install` | `yarn` | `pnpm install` |
| `npm install pkg` | `yarn add pkg` | `pnpm add pkg` |
| `npm install -D pkg` | `yarn add -D pkg` | `pnpm add -D pkg` |
| `npm run dev` | `yarn dev` | `pnpm dev` |
| `npm run build` | `yarn build` | `pnpm build` |

## Laravel Project Setup

### package.json

```json
{
    "private": true,
    "type": "module",
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview"
    },
    "devDependencies": {
        "@tailwindcss/vite": "^4.0.0",
        "axios": "^1.7.4",
        "laravel-vite-plugin": "^1.0.0",
        "tailwindcss": "^4.0.0",
        "vite": "^6.0.0"
    }
}
```

### vite.config.js (Laravel + Tailwind v4)

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
});
```

### vite.config.js (Laravel + Filament Theme)

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
                'resources/css/filament/admin/theme.css',
            ],
            refresh: true,
        }),
        tailwindcss(),
    ],
});
```

## Tailwind CSS v4 Setup

```bash
# Install Tailwind v4
pnpm add -D tailwindcss@next @tailwindcss/vite@next

# Install plugins (if needed)
pnpm add -D @tailwindcss/forms @tailwindcss/typography
```

**resources/css/app.css:**
```css
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
}
```

## React Setup (Inertia)

```bash
pnpm add @inertiajs/react react react-dom
pnpm add -D @vitejs/plugin-react @types/react @types/react-dom
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.tsx',
            refresh: true,
        }),
        react(),
    ],
});
```

## Vue Setup (Inertia)

```bash
pnpm add @inertiajs/vue3 vue
pnpm add -D @vitejs/plugin-vue
```

```js
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
        vue(),
    ],
});
```

## Troubleshooting

### Clear cache and reinstall

```bash
rm -rf node_modules
rm -rf node_modules/.vite
pnpm install
```

### Peer dependency issues

```bash
pnpm install --shamefully-hoist
```

### Hot reload not working

```js
// vite.config.js
export default defineConfig({
    server: {
        hmr: {
            host: 'localhost',
        },
    },
    // ...
});
```

### Build fails in production

```bash
# Ensure all dependencies are installed
pnpm install --prod=false
pnpm build
```

## CI/CD (GitHub Actions)

```yaml
- name: Setup pnpm
  uses: pnpm/action-setup@v2
  with:
    version: 9

- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'pnpm'

- name: Install dependencies
  run: pnpm install --frozen-lockfile

- name: Build assets
  run: pnpm build
```

## Monorepo (Workspace)

**pnpm-workspace.yaml:**
```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

```bash
# Run in specific package
pnpm --filter package-name dev

# Run in all packages
pnpm -r build

# Add dependency to specific package
pnpm --filter package-name add lodash
```

## Reference Files

- `~/.claude/skills/environment/pnpm-node.md`
