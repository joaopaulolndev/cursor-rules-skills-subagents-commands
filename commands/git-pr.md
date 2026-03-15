# Criar Pull Request

Crie um Pull Request bem documentado para a branch atual.

## Passos

1. Execute `git log main..HEAD --oneline` para ver os commits do PR
2. Execute `git diff main...HEAD --stat` para ver arquivos alterados
3. Identifique o tipo de mudança (feature, fix, refactor, etc.)
4. Gere o título do PR seguindo: `tipo: descrição curta`
5. Gere o body do PR com esta estrutura:

```markdown
## O que foi feito
Descrição clara do que foi implementado/corrigido.

## Por que foi feito
Contexto e motivação da mudança.

## Como testar
1. Passo 1
2. Passo 2

## Checklist
- [ ] Testes adicionados/atualizados
- [ ] Sem N+1 queries
- [ ] Migrations com up() e down()
- [ ] Sem dados sensíveis expostos
```

6. Execute `gh pr create --title "título" --body "body"` ou exiba o conteúdo para copiar
