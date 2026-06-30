---
name: "speckit-taskstoissues"
description: "Converter tarefas existentes em issues acionáveis do GitHub, ordenadas por dependência, para a feature com base nos artefatos de design disponíveis."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/taskstoissues.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da conversão de tarefas em issues)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_taskstoissues`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):
    ```
    ## Hooks de Extensão

    **Pré-Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```
  - **Hook obrigatório** (`optional: false`):
    ```
    ## Hooks de Extensão

    **Pré-Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}

    Aguarde o resultado do comando do hook antes de prosseguir para o Roteiro.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Roteiro

1. Execute `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` a partir da raiz do repositório e analise FEATURE_DIR e a lista AVAILABLE_DOCS. Todos os caminhos devem ser absolutos. Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").
1. **SE EXISTIR**: Carregue `.specify/memory/constitution.md` para princípios do projeto e restrições de governança.
1. Do script executado, extraia o caminho para **tasks**.
1. Obtenha o remote Git executando:

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> PROSSIGA PARA AS PRÓXIMAS ETAPAS APENAS SE O REMOTE FOR UMA URL DO GITHUB

1. **Buscar issues existentes para deduplicação**: Antes de criar qualquer coisa, construa o conjunto de IDs de tarefa que você está prestes a processar a partir de `tasks.md` (cada um é um `T` seguido de três dígitos, ex. `T001`). Em seguida, use a ferramenta `list_issues` do servidor MCP do GitHub para procurar issues que já cubram esses IDs. Não passe um valor `state`, pois omiti-lo faz a ferramenta retornar issues abertas e fechadas. Solicite `perPage: 100` para reduzir o número de chamadas e, como a ferramenta usa paginação baseada em cursor, solicite páginas com o parâmetro `after` (usando o `endCursor` da resposta anterior). Para cada título de issue, compare-o com o padrão de ID de tarefa `\bT\d{3}\b` (limites de palavra para que tokens como `ST001` ou `T0010` não sejam correspondidos por engano; isso também reconhece títulos escritos como `T001 ...`, `T001: ...` ou `[T001] ...`) e, quando corresponder a um dos seus IDs de tarefa, marque esse ID como já tendo uma issue. Pare de paginar assim que cada ID de tarefa tiver sido correspondido, ou quando não houver mais páginas, para não continuar buscando todo o histórico de issues do repositório depois que todos os IDs de tarefa estiverem contabilizados. Isso limita o número de chamadas em repositórios com históricos grandes de issues e ainda previne duplicatas quando o comando é reexecutado após `tasks.md` ser regenerado ou a skill ser reinvocada.
1. Para cada tarefa na lista, use o servidor MCP do GitHub para criar uma nova issue no repositório representativo do remote Git. Linhas de tarefa em `tasks.md` começam com um checkbox markdown, então primeiro remova o `- [ ]` inicial (e quaisquer marcadores `[P]` / `[US#]`) para recuperar o ID da tarefa e sua descrição. Crie a issue com um único título canônico no formato `T001: <descrição>`, com o ID escrito uma vez seguido da descrição da tarefa (por exemplo, a linha `- [ ] T001 Criar estrutura do projeto` torna-se o título `T001: Criar estrutura do projeto`).
   - **Pule** qualquer tarefa cujo ID já esteja presente no conjunto de issues existentes da etapa anterior, e reporte (por exemplo, `T001 já possui uma issue, pulando`).
   - Crie issues apenas para tarefas que ainda não tenham uma issue correspondente.

> [!CAUTION]
> EM HIPÓTESE ALGUMA CRIE ISSUES EM REPOSITÓRIOS QUE NÃO CORRESPONDAM À URL DO REMOTE

## Verificações Pós-Execução

**Verificar hooks de extensão (após conversão de tarefas em issues)**:
Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_taskstoissues`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):
    ```
    ## Hooks de Extensão

    **Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```
  - **Hook obrigatório** (`optional: false`):
    ```
    ## Hooks de Extensão

    **Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente
