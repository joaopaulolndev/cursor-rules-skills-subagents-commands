# Git Commit

Analise todas as mudanças staged (`git diff --cached`) e gere um commit seguindo Conventional Commits.

## Passos

1. Execute `git diff --cached` para ver o que está staged
2. Se nada estiver staged, execute `git status` e pergunte o que incluir
3. Analise as mudanças e escolha o tipo correto:
   - `feat`: nova funcionalidade
   - `fix`: correção de bug
   - `refactor`: refatoração sem mudança de comportamento
   - `chore`: tarefas de manutenção (deps, config)
   - `docs`: documentação
   - `test`: testes
   - `perf`: performance
   - `style`: formatação, pint, sem mudança de lógica

4. Gere a mensagem no formato:
   ```
   tipo(escopo): descrição curta em português
   
   - detalhe 1
   - detalhe 2
   ```

5. Execute o commit com `git commit -m "mensagem"`

## Regras

- Descrição em letras minúsculas
- Máximo 72 caracteres na primeira linha
- Escopo é opcional mas recomendado (ex: `feat(auth):`, `fix(api):`)
- Se houver breaking change, adicione `!` após o tipo: `feat(api)!:`
