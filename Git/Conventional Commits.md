# Conventional commits
Boa — segue um resumo objetivo das principais flags do `git commit` com quando usá‑las e exemplos usando o padrão Conventional Commits, seguido de uma tabela com os tipos mais usados do Conventional Commits.

| Flag                          | O que faz                                                                                              | Quando usar / Exemplo (Conventional Commits)                                                                                                                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-m "<mensagem>"`             | Define a mensagem do commit inline.                                                                    | Use para commits simples. Ex: `git commit -m "feat(auth): adicionar login via OAuth"`                                                                                                                         |
| `-m "<linha1>" -m "<linha2>"` | Permite múltiplas linhas (cada `-m` vira um parágrafo).                                                | Quando precisar de resumo + corpo + footer (BREAKING CHANGE). Ex: `git commit -m "feat(api): nova rota /users" -m "Adiciona paginação e filtros." -m "BREAKING CHANGE: query param 'page' agora obrigatório"` |
| `-e` / `--edit`               | Abre o editor para editar a mensagem do commit.                                                        | Quando quer escrever/formatar a mensagem manualmente (útil para corpo/footers). `git commit -e`                                                                                                               |
| `-a` / `--all`                | Adiciona automaticamente arquivos rastreados modificados (faz `git add` para arquivos já versionados). | Para commitar todas as mudanças em arquivos já rastreados. Ex: `git commit -a -m "fix(ui): corrigir overflow no header"`                                                                                      |
| `--amend`                     | Altera o último commit (mensagem e/ou conteúdo).                                                       | Corrigir mensagem ou incluir arquivos esquecidos. Ex: `git commit --amend -m "docs: ajustar README"` (atenção: reescreve histórico)<br>`git commit --amend --no-edit`                                         |
| `-s` / `--signoff`            | Adiciona uma linha `Signed-off-by:` na mensagem.                                                       | Quando é exigido (DCO) ou se deseja declarar autoria. `git commit -s -m "chore: atualizar dependências"`                                                                                                      |
| `-S` / `--gpg-sign`           | Assina o commit com GPG.                                                                               | Para garantir integridade/autenticidade. `git commit -S -m "feat:..."`                                                                                                                                        |
| `--no-verify`                 | Pula hooks pre-commit e commit-msg.                                                                    | Usar com cuidado (p.ex. para bypassar checks locais). Não recomendado para contornar verificação de mensagens de Conventional Commits. `git commit --no-verify -m "fix:..."`                                  |
| `--allow-empty`               | Cria um commit mesmo sem alterações.                                                                   | Útil para criar tags/releases ou registrar trabalho sem mudanças. Ex: `git commit --allow-empty -m "chore(release): 1.2.0"`                                                                                   |
| `-F <arquivo>`                | Lê a mensagem de commit de um arquivo.                                                                 | Quando gerar mensagens automaticamente (CI) ou ter template pronto. `git commit -F commit_msg.txt`                                                                                                            |
| `--author="Nome <email>"`     | Define autor diferente do configurado.                                                                 | Para commitar em nome de outro autor (ex.: correção de commit). `git commit --author="Maria <maria@ex.com>" -m "fix:..."`                                                                                     |
| `--date="<data>"`             | Define data do commit.                                                                                 | Para scripts/CI que precisam de datas específicas. `git commit --date="2025-08-28T09:00:00-03:00" -m "chore:..."`                                                                                             |
| `-v` / `--verbose`            | Mostra diff no editor de mensagem.                                                                     | Ao escrever mensagem, ver contexto das mudanças. `git commit -v`                                                                                                                                              |

Tabela de tipos principais do Conventional Commits (quando usar)

| Tipo | Descrição | Exemplo de subject |
|---|---:|---|
| feat | Nova funcionalidade | `feat(auth): adicionar login via OAuth` |
| fix | Correção de bug | `fix(api): corrigir retorno 500 em /orders` |
| docs | Mudanças na documentação | `docs: atualizar README com instruções de setup` |
| style | Formatação/estilo sem afetar lógica (espaços, lint) | `style: ajustar indentação em components/` |
| refactor | Mudança de código que não adiciona feature nem corrige bug | `refactor(auth): extrair validação para service` |
| perf | Melhorias de performance | `perf(db): otimizar query de relatórios` |
| test | Adição/ajuste de testes | `test: adicionar testes unitários para OrderService` |
| build | Mudanças no sistema de build/tooling | `build: atualizar pipeline do CI para Node 18` |
| ci | Mudanças de configuração do CI | `ci: adicionar job de lint` |
| chore | Tarefas de manutenção sem alterar src (deps, configs) | `chore: atualizar dependências` |
| revert | Reverte um commit anterior | `revert: feat(auth): adicionar login via OAuth` |
| BREAKING CHANGE | Footer para mudanças incompatíveis | No body/footer: `BREAKING CHANGE: API /v1/users removida` |

Boas práticas rápidas
- Use scope quando fizer sentido: `feat(login): ...` (ajuda em changelogs).  
- Resuma no subject em até ~50 caracteres e sem ponto final: `tipo(scope): resumo`.  
- Use corpo para explicar o quê, por que e qual a alternativa.  
- Se houver quebra de compatibilidade, inclua `BREAKING CHANGE: descrição` no footer (ou prefixo `!` no subject, ex: `feat!: mudança que quebra API`).  
- Para corrigir uma mensagem depois de commitar: `git commit --amend -m "fix: mensagem corrigida"` (evite em branches compartilhadas sem força consciente).

Se quiser, eu gero templates de commit ou um hook `commit-msg` simples para validar Conventional Commits automaticamente. Quer que eu faça isso?