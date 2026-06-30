---
name: "speckit-converge"
description: "Avaliar o codebase atual contra a spec, plan e tasks da feature, e acrescentar qualquer trabalho restante não construído como novas tarefas em tasks.md para que implement possa completá-lo."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/converge.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da convergência)**:

- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_converge`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):

    ```text
    ## Hooks de Extensão

    **Pré-Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```

  - **Hook obrigatório** (`optional: false`):

    ```text
    ## Hooks de Extensão

    **Pré-Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}

    Aguarde o resultado do comando do hook antes de prosseguir para o Objetivo.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.

- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Objetivo

Fechar a lacuna entre o que a especificação, plano e tarefas de uma feature exigem e o que o
codebase atualmente implementa. Leia `spec.md`, `plan.md` e `tasks.md` como a **única
fonte de intenção** (com a constitution como restrições governantes), avalie o estado atual
do código, determine quais requisitos, critérios de aceitação, decisões de plano e
tarefas existentes estão não atendidos, incompletos ou apenas parcialmente satisfeitos, e **acrescente cada pedaço
de trabalho restante como uma nova tarefa rastreável** no final de `tasks.md` para que
`/speckit-implement` possa completá-lo. Este comando DEVE executar apenas após
`/speckit-implement` ter executado no `tasks.md` atual, e após `/speckit-tasks` ter produzido um `tasks.md` completo.

Isto **não** é uma ferramenta de diff e **não** rastreia mudanças. Avalia o estado presente
do código em relação aos artefatos da feature — sem git, sem comparação de branch, sem histórico.

## Restrições Operacionais

**SOMENTE ACRÉSCIMO, NUNCA REESCREVER**: A **única** escrita do comando é acrescentar uma nova
seção `## Fase N: Convergência` a `tasks.md`. Ele NÃO DEVE:

- modificar `spec.md` ou `plan.md` de qualquer forma;
- reescrever, renumerar, reordenar ou deletar qualquer tarefa existente (incluindo tarefas de uma fase
  Convergência anterior);
- modificar, criar ou deletar qualquer código de aplicação — completar as tarefas acrescentadas é o
  trabalho de `/speckit-implement`.

Quando o codebase já satisfaz tudo, o comando DEVE deixar `tasks.md`
**inalterado byte a byte** (sem cabeçalho Convergência vazio) e reportar um resultado limpo.

**Autoridade da Constitution**: A constitution do projeto (`.specify/memory/constitution.md`) é
**não negociável**. Código que viola um princípio MUST é a descoberta de maior severidade e
produz uma tarefa de remediação correspondente. Se a constitution for um template não preenchido,
pule verificações da constitution graciosamente em vez de falhar.

## Etapas de Execução

### 1. Inicializar Contexto de Convergência

Execute `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` uma vez a partir da raiz do repositório e analise o JSON para FEATURE_DIR e AVAILABLE_DOCS. Derive caminhos absolutos:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md
- CONSTITUTION = `.specify/memory/constitution.md` (se presente)
Se `spec.md`, `plan.md` ou `tasks.md` estiver ausente, PARE com uma mensagem clara e acionável nomeando o
comando pré-requisito a executar (`/speckit-specify` para spec ausente, `/speckit-plan` para plan ausente,
`/speckit-tasks` para tarefas ausentes). Não produza saída parcial.
Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

### 2. Carregar Artefatos (Divulgação Progressiva)

Carregue apenas o contexto mínimo necessário de cada artefato:

**De spec.md:**

- Requisitos Funcionais (FR-###)
- Critérios de Sucesso (SC-###) — inclua apenas itens que exijam trabalho construível; exclua
  métricas de resultado pós-lançamento e KPIs de negócio
- User Stories e seus Cenários de Aceitação
- Casos Extremos (se presentes)

**De plan.md:**

- Escolhas de arquitetura/stack e decisões técnicas
- Referências do Modelo de Dados
- Fases e pontos de contato nomeados (arquivos/componentes que o plano diz que serão criados ou editados)
- Restrições técnicas

**De tasks.md:**

- IDs de tarefa (para calcular o próximo ID e próximo número de fase)
- Descrições, agrupamento por fase e caminhos de arquivo referenciados

**Da constitution (se não for template não preenchido):**

- Nomes de princípios e declarações normativas MUST/SHOULD

### 3. Construir o Inventário de Intenção

Crie um modelo interno (não ecoe artefatos brutos):

- **Inventário de requisitos**: uma chave estável por FR-### / SC-### / cenário de aceitação
  de user story (ex. `US1/AC2`), mais as decisões de plano e princípios da constitution que
  impõem obrigações construíveis.
- **Mapa de escopo do código**: a partir dos caminhos de arquivo nomeados em `plan.md` e `tasks.md`, mais uma busca
  por palavra-chave dos conceitos que cada requisito descreve, derive o conjunto de arquivos fonte e
  componentes no escopo para avaliação. Limite a avaliação a estes — **não** infira
  escopo além do que os artefatos definem.

### 4. Avaliar o Codebase e Classificar Descobertas

Para cada item no inventário de intenção, inspecione o código atual no escopo e produza um
`Finding` apenas onde houver uma lacuna. Classifique cada descoberta por **tipo de lacuna**:

- **`ausente`**: o trabalho exigido está ausente do código inteiramente.
- **`parcial`**: o trabalho existe mas ainda não satisfaz totalmente o requisito /
  critério de aceitação / decisão de plano.
- **`contradiz`**: o código faz algo que conflita com a intenção declarada ou um
  princípio MUST da constitution.
- **`não solicitada`**: o código contém trabalho não solicitado pela spec, plan ou tasks
  (exposto para consciência — converge **não** deleta código, apenas acrescenta uma tarefa para
  revisar/justificar ou removê-lo).

Cada `Finding` registra: um id estável, o `source-ref` ao qual rastreia, o `gap-type`, uma
severidade e uma descrição curta legível com a evidência (o arquivo/área observada).

**Casos extremos:**

- **Pouco ou nenhum código ainda**: trate todo o escopo especificado como trabalho restante `ausente`
  em vez de falhar.
- **Nada resta**: produza zero descobertas e siga o ramo convergido na Etapa 7.

### 5. Atribuir Severidade

- **CRÍTICO**: viola um princípio MUST da constitution, ou uma lacuna `ausente`/`contradiz`
  que bloqueia funcionalidade base de uma user story P1.
- **ALTA**: uma lacuna `ausente` ou `parcial` em um requisito funcional core ou critério de aceitação.
- **MÉDIA**: uma lacuna `parcial` em um requisito secundário, ou uma adição `não solicitada` com
  justificativa pouco clara.
- **BAIXA**: lacunas parciais menores, polimento ou adições `não solicitadas` de baixo risco.

### 6. Apresentar o Resumo de Descobertas na Sessão

Antes de acrescentar qualquer coisa, produza um resumo compacto graduado por severidade (sem escrita em arquivo ainda):

## Descobertas de Convergência

| ID | Tipo de Lacuna | Severidade | Origem | Evidência | Trabalho Restante |
|----|----------|----------|--------|----------|----------------|
| F1 | ausente  | ALTA     | FR-008 | Exemplo: nenhuma proteção somente-acréscimo detectada em path/to/module.py ao escrever tasks.md | Adicionar imposição de somente-acréscimo |

**Métricas de resumo:**

- Requisitos / critérios de aceitação verificados
- Decisões de plano verificadas
- Princípios da constitution verificados (ou "ignorado — template")
- Descobertas por tipo de lacuna (ausente / parcial / contradiz / não solicitada)
- Descobertas por severidade

### 7. Acrescentar Tarefas de Convergência (ou reportar convergido)

**Se houver uma ou mais descobertas acionáveis** (resultado `tasks_appended`):

Acrescente ao **final** de `tasks.md`, conforme o contrato de acréscimo:

1. Escaneie todos os IDs de tarefa existentes; seja `M` o máximo. Determine o próximo número de fase `N`
   (maior fase existente + 1).
2. Escreva um único cabeçalho de seção `## Fase N: Convergência`.
3. Emita um item de checklist por descoberta acionável, ordenados CRÍTICO/ALTA primeiro, atribuindo
   IDs com zero-padding `T{M+1:03d}, T{M+2:03d}, …`:

   ```markdown
   - [ ] T042 <descrição imperativa> conforme <source-ref> (<gap-type>)
   ```

   `<source-ref>` rastreia a tarefa à sua origem: ex. `FR-003`, `SC-002`,
   `US1/AC2`, `plan: decisão de armazenamento`, `Constitution II`.

   `<gap-type>` é um de `ausente`, `parcial`, `contradiz`, `não solicitada`.

   Tarefas de violação da constitution DEVEM ser emitidas primeiro e descritas como
   `CRÍTICO`.
4. Nunca reutilize ou renumerar IDs existentes. Se uma fase Convergência anterior existir, adicione uma nova,
   separadamente numerada abaixo dela — não toque na antiga.

**Se não houver descobertas acionáveis** (resultado `converged`):

- **Não** modifique `tasks.md` de forma alguma — sem cabeçalho de fase vazio.
- Reporte: **"✅ Convergido — a implementação satisfaz a spec, o plano e as tarefas."**
- Inclua as contagens de resumo do que foi verificado.

### 8. Fornecer Próximas Ações (Repasse)

- Em `tasks_appended`: informe quantas tarefas foram acrescentadas sob qual fase, e recomende
  executar `/speckit-implement` para completá-las; observe que uma execução de converge
  de acompanhamento encontrará menos ou nenhum item restante.
- Em `converged`: recomende prosseguir para revisão / abrir um PR. Nenhuma passagem adicional de implement é necessária para o escopo especificado desta feature.

### 9. Verificar hooks de extensão

Após produzir o resultado, verifique se `.specify/extensions.yml` existe na raiz do projeto.

- Se existir, leia-o e procure entradas sob a chave `hooks.after_converge`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Reporte o resultado da convergência (`converged` ou `tasks_appended`) na sessão antes de listar
  quaisquer hooks, para que os usuários possam decidir se executam comandos de acompanhamento opcionais.
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):

    ```text
    ## Hooks de Extensão

    **Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```

  - **Hook obrigatório** (`optional: false`):

    ```text
    ## Hooks de Extensão

    **Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.

- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente
