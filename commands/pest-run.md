# Pest Run

Execute os testes Pest com as opções corretas para o contexto atual.

## Passos

1. Verifique o arquivo/contexto aberto para decidir o escopo:
   - Em arquivo de teste → rode só esse arquivo
   - Em controller/model → rode o teste correspondente
   - Sem contexto → rode a suite completa

2. Escolha o comando adequado:

```bash
# Suite completa
./vendor/bin/pest

# Arquivo específico
./vendor/bin/pest tests/Feature/Posts/CreatePostTest.php

# Por filtro (nome do teste)
./vendor/bin/pest --filter="can create post"

# Por grupo
./vendor/bin/pest --group=feature

# Paralelo (mais rápido)
./vendor/bin/pest --parallel

# Para no primeiro erro
./vendor/bin/pest --stop-on-failure

# Com coverage
./vendor/bin/pest --coverage --min=80
```

3. Analise os erros e sugira correções se os testes falharem
4. Se usar Laravel Herd no Windows: substitua `./vendor/bin/pest` por `"$HOME/.config/herd/bin/php.bat" vendor/bin/pest`
