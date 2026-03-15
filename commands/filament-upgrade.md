# Filament Upgrade

Guie o processo de upgrade do Filament para a próxima versão major.

## Passos

1. Verifique a versão atual:
```bash
composer show filament/filament | grep versions
```

2. Identifique a versão atual e o caminho de upgrade:
   - **v3 → v4:** requer Tailwind v4 e PHP 8.2+
   - **v4 → v5:** requer Livewire v4

3. Verifique pré-requisitos:
```bash
php -v
composer show laravel/framework | grep versions
```

4. Execute o upgrade:
```bash
composer require filament/filament:"^VERSAO" --update-with-dependencies
php artisan filament:upgrade
```

5. Verifique breaking changes críticos por versão:

**v3 → v4:**
- `Filament\Forms\Form` → `Filament\Schemas\Schema`
- `Filament\Tables\Table` mantém, mas imports mudam
- Tailwind config migra de `tailwind.config.js` para CSS nativo

**v4 → v5:**
- Livewire v3 → v4 obrigatório
- Verificar componentes Livewire customizados

6. Rode os testes: `php artisan test`
7. Limpe caches: `php artisan optimize:clear`
