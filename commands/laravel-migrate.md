# Laravel Migrate

Execute migrations com segurança verificando o ambiente antes.

## Passos

1. Verifique o ambiente atual com `php artisan env`
2. Mostre as migrations pendentes com `php artisan migrate:status`
3. Se estiver em **produção**, pare e avise o usuário — não execute sem confirmação explícita
4. Em desenvolvimento, execute `php artisan migrate`
5. Se houver erro, mostre o erro completo e sugira solução

## Variações — pergunte ao usuário qual deseja

- **Rollback último batch:** `php artisan migrate:rollback`
- **Resetar tudo:** `php artisan migrate:fresh` (apaga todos os dados!)
- **Com seeders:** `php artisan migrate:fresh --seed`
- **Rollback N steps:** `php artisan migrate:rollback --step=N`

## Aviso importante

Antes de `migrate:fresh` em qualquer ambiente, confirme que há backup ou que a perda de dados é intencional.
