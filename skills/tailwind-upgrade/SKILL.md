---
description: Upgrade Tailwind CSS v3 to v4. Handles config migration from JS to CSS, Vite setup, and Filament integration.
---

# Tailwind CSS v4 Upgrade Assistant

You are a Tailwind CSS upgrade specialist. Help migrate projects from v3 to v4.

## Important

Tailwind v4 is **REQUIRED** for:
- Filament v4+
- Modern Laravel projects

## Major Changes in v4

1. **No more `tailwind.config.js`** - Configuration via CSS
2. **No more `@tailwind` directives** - Use `@import "tailwindcss"`
3. **PostCSS not required** for Vite projects
4. **Automatic content detection**

## Migration Steps

### 1. Update Dependencies

```bash
# Using pnpm
pnpm add -D tailwindcss@next @tailwindcss/vite@next
pnpm remove autoprefixer  # Not needed anymore

# Using npm
npm install -D tailwindcss@next @tailwindcss/vite@next
npm uninstall autoprefixer
```

### 2. Update vite.config.js

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

### 3. Convert tailwind.config.js to CSS

**Before (v3 - tailwind.config.js):**
```js
module.exports = {
    content: [
        './resources/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
    ],
    theme: {
        extend: {
            colors: {
                primary: {
                    50: '#eff6ff',
                    500: '#3b82f6',
                    600: '#2563eb',
                },
            },
            fontFamily: {
                sans: ['Inter', 'sans-serif'],
            },
        },
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
}
```

**After (v4 - resources/css/app.css):**
```css
@import "tailwindcss";
@import "@tailwindcss/forms";
@import "@tailwindcss/typography";

@source "../views/**/*.blade.php";
@source "../../vendor/filament/**/*.blade.php";

@theme {
  --color-primary-50: #eff6ff;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --font-sans: "Inter", sans-serif;
}
```

### 4. Remove Old Files

```bash
rm tailwind.config.js
rm postcss.config.js  # Not needed for Vite
```

### 5. Update CSS Imports

**Before (v3):**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

**After (v4):**
```css
@import "tailwindcss";
```

## Filament Theme Integration

For Filament v4/v5 custom themes:

**resources/css/filament/admin/theme.css:**
```css
@import "tailwindcss";
@import "../../../../vendor/filament/filament/resources/css/theme.css";

@source "../../../../app/Filament/**/*.php";
@source "../../../../resources/views/filament/**/*.blade.php";
@source "../../../../vendor/filament/**/*.blade.php";

@theme {
  --color-primary-50: #eff6ff;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;

  --color-danger-500: #ef4444;
  --color-success-500: #22c55e;
  --color-warning-500: #f59e0b;
}
```

**Register in PanelProvider:**
```php
public function panel(Panel $panel): Panel
{
    return $panel
        ->viteTheme('resources/css/filament/admin/theme.css');
}
```

## Class Changes

| v3 | v4 |
|----|-----|
| `bg-opacity-50` | `bg-black/50` |
| `text-opacity-75` | `text-white/75` |
| `bg-gradient-to-r` | `bg-linear-to-r` |

## Build Commands

```bash
# Development
pnpm dev

# Production
pnpm build
```

## Troubleshooting

### Classes not applied
1. Check `@source` paths are correct
2. Run `pnpm build` to rebuild
3. Clear browser cache

### Custom colors not working
```css
/* Wrong */
--my-color: blue;

/* Correct - must be inside @theme */
@theme {
  --color-my-color: blue;
}
```

## Reference Files

- `~/.claude/skills/tailwindcss/upgrade-v3-to-v4.md`
