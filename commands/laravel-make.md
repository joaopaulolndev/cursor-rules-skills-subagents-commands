# Laravel Make

Gere arquivos Laravel com os comandos artisan corretos.

## Como usar

Descreva o que precisa criar. Exemplos:
- "model Post com migration, factory e controller"
- "form request para validar criação de pedido"
- "job para enviar email de boas-vindas"

## Mapeamento de geradores

| O que criar | Comando |
|---|---|
| Model + tudo | `php artisan make:model Nome -mfsc --policy` |
| Só migration | `php artisan make:migration create_tabela_table` |
| Controller resource | `php artisan make:controller NomesController --resource` |
| API Controller | `php artisan make:controller Api/V1/NomesController --api` |
| Form Request | `php artisan make:request StoreNomeRequest` |
| API Resource | `php artisan make:resource NomeResource` |
| Job | `php artisan make:job NomeJob` |
| Event | `php artisan make:event NomeEvent` |
| Listener | `php artisan make:listener NomeListener --event=NomeEvent` |
| Policy | `php artisan make:policy NomePolicy --model=Nome` |
| Observer | `php artisan make:observer NomeObserver --model=Nome` |
| Command | `php artisan make:command NomeCommand` |
| Middleware | `php artisan make:middleware NomeMiddleware` |
| Seeder | `php artisan make:seeder NomeSeeder` |
| Factory | `php artisan make:factory NomeFactory --model=Nome` |

## Passos

1. Identifique o que o usuário quer criar
2. Execute o(s) comando(s) correto(s)
3. Mostre os arquivos criados e o próximo passo sugerido
