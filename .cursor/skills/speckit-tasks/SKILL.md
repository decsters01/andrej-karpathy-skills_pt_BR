---
name: "speckit-tasks"
description: "Gerar um tasks.md acionável e ordenado por dependência para a feature com base nos artefatos de design disponíveis."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/tasks.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da geração de tarefas)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_tasks`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):
    ```
    ## Extension Hooks

    **Pre-Hook Opcional**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    Para executar: `/{command}`
    ```
  - **Hook obrigatório** (`optional: false`):
    ```
    ## Extension Hooks

    **Pre-Hook Automático**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    
    Aguarde o resultado do comando do hook antes de prosseguir para o Roteiro.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Roteiro

1. **Configuração**: Execute `.specify/scripts/bash/setup-tasks.sh --json` a partir da raiz do repositório e analise FEATURE_DIR, TASKS_TEMPLATE e a lista AVAILABLE_DOCS. `FEATURE_DIR` e `TASKS_TEMPLATE` devem ser caminhos absolutos quando fornecidos. `AVAILABLE_DOCS` é uma lista de nomes de documentos/caminhos relativos disponíveis sob `FEATURE_DIR` (por exemplo `research.md` ou `contracts/`). Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

2. **Carregar documentos de design**: Leia de FEATURE_DIR:
   - **Obrigatório**: plan.md (stack técnica, bibliotecas, estrutura), spec.md (user stories com prioridades)
   - **Opcional**: data-model.md (entidades), contracts/ (contratos de interface), research.md (decisões), quickstart.md (cenários de teste)
   - **SE EXISTIR**: Carregue `.specify/memory/constitution.md` para princípios do projeto e restrições de governança
   - Nota: Nem todos os projetos têm todos os documentos. Gere tarefas com base no que estiver disponível.

3. **Executar fluxo de geração de tarefas**:
   - Carregue plan.md e extraia stack técnica, bibliotecas, estrutura do projeto
   - Carregue spec.md e extraia user stories com suas prioridades (P1, P2, P3, etc.)
   - Se data-model.md existir: Extraia entidades e mapeie para user stories
   - Se contracts/ existir: Mapeie contratos de interface para user stories
   - Se research.md existir: Extraia decisões para tarefas de setup
   - Gere tarefas organizadas por user story (veja Regras de Geração de Tarefas abaixo)
   - Gere grafo de dependências mostrando ordem de conclusão das user stories
   - Crie exemplos de execução paralela por user story
   - Valide completude das tarefas (cada user story tem todas as tarefas necessárias, testável independentemente)

4. **Gerar tasks.md**: Leia o template de tarefas de TASKS_TEMPLATE (da saída JSON acima) e use-o como estrutura. Se TASKS_TEMPLATE estiver vazio, use `.specify/templates/tasks-template.md` como fallback. Preencha com:
   - Nome correto da feature de plan.md
   - Fase 1: Tarefas de Setup (inicialização do projeto)
   - Fase 2: Tarefas Fundacionais (pré-requisitos bloqueantes para todas as user stories)
   - Fase 3+: Uma fase por user story (em ordem de prioridade de spec.md)
   - Cada fase inclui: objetivo da story, critérios de teste independente, testes (se solicitados), tarefas de implementação
   - Fase Final: Polimento e preocupações transversais
   - Todas as tarefas devem seguir o formato estrito de checklist (veja Regras de Geração de Tarefas abaixo)
   - Caminhos de arquivo claros para cada tarefa
   - Seção de dependências mostrando ordem de conclusão das stories
   - Exemplos de execução paralela por story
   - Seção de estratégia de implementação (MVP primeiro, entrega incremental)

## Hooks Obrigatórios Pós-Execução

**Você DEVE completar esta seção antes de reportar a conclusão ao usuário.**

Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se não existir, ou nenhum hook estiver registrado sob `hooks.after_tasks`, pule para o Relatório de Conclusão.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_tasks`.
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue para o Relatório de Conclusão.
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook obrigatório** (`optional: false`) — **Você DEVE emitir `EXECUTE_COMMAND:` para cada hook obrigatório**:
    ```
    ## Extension Hooks

    **Hook Automático**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
  - **Hook opcional** (`optional: true`):
    ```
    ## Extension Hooks

    **Hook Opcional**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    Para executar: `/{command}`
    ```

## Relatório de Conclusão

Produza o caminho do tasks.md gerado e resumo:
- Contagem total de tarefas
- Contagem de tarefas por user story
- Oportunidades paralelas identificadas
- Critérios de teste independente para cada story
- Escopo MVP sugerido (tipicamente apenas User Story 1)
- Validação de formato: Confirme que TODAS as tarefas seguem o formato de checklist (checkbox, ID, labels, caminhos de arquivo)

Contexto para geração de tarefas: $ARGUMENTS

O tasks.md deve ser imediatamente executável — cada tarefa deve ser específica o suficiente para que um LLM possa completá-la sem contexto adicional.

## Regras de Geração de Tarefas

**CRÍTICO**: Tarefas DEVEM ser organizadas por user story para permitir implementação e teste independentes.

**Testes são OPCIONAIS**: Gere tarefas de teste apenas se explicitamente solicitado na especificação da feature ou se o usuário solicitar abordagem TDD.

### Formato de Checklist (OBRIGATÓRIO)

Cada tarefa DEVE seguir estritamente este formato:

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**Componentes do Formato**:

1. **Checkbox**: SEMPRE comece com `- [ ]` (checkbox markdown)
2. **Task ID**: Número sequencial (T001, T002, T003...) em ordem de execução
3. **Marcador [P]**: Inclua APENAS se a tarefa for paralelizável (arquivos diferentes, sem dependências de tarefas incompletas)
4. **Label [Story]**: OBRIGATÓRIO apenas para tarefas de fase de user story
   - Formato: [US1], [US2], [US3], etc. (mapeia para user stories de spec.md)
   - Fase Setup: SEM label de story
   - Fase Fundacional: SEM label de story  
   - Fases de User Story: DEVE ter label de story
   - Fase Polimento: SEM label de story
5. **Descrição**: Ação clara com caminho exato de arquivo

**Exemplos**:

- ✅ CORRETO: `- [ ] T001 Create project structure per implementation plan`
- ✅ CORRETO: `- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py`
- ✅ CORRETO: `- [ ] T012 [P] [US1] Create User model in src/models/user.py`
- ✅ CORRETO: `- [ ] T014 [US1] Implement UserService in src/services/user_service.py`
- ❌ ERRADO: `- [ ] Create User model` (falta ID e label Story)
- ❌ ERRADO: `T001 [US1] Create model` (falta checkbox)
- ❌ ERRADO: `- [ ] [US1] Create User model` (falta Task ID)
- ❌ ERRADO: `- [ ] T001 [US1] Create model` (falta caminho de arquivo)

### Organização de Tarefas

1. **Das User Stories (spec.md)** - ORGANIZAÇÃO PRIMÁRIA:
   - Cada user story (P1, P2, P3...) recebe sua própria fase
   - Mapeie todos os componentes relacionados à sua story:
     - Models necessários para essa story
     - Services necessários para essa story
     - Interfaces/UI necessárias para essa story
     - Se testes solicitados: Testes específicos para essa story
   - Marque dependências entre stories (a maioria das stories deve ser independente)

2. **Dos Contratos**:
   - Mapeie cada contrato de interface → para a user story que ele serve
   - Se testes solicitados: Cada contrato de interface → tarefa de teste de contrato [P] antes da implementação na fase dessa story

3. **Do Modelo de Dados**:
   - Mapeie cada entidade para a(s) user story(ies) que a necessitam
   - Se entidade serve múltiplas stories: Coloque na story mais antiga ou na fase Setup
   - Relacionamentos → tarefas de camada de service na fase apropriada da story

4. **Do Setup/Infraestrutura**:
   - Infraestrutura compartilhada → Fase Setup (Fase 1)
   - Tarefas fundacionais/bloqueantes → Fase Fundacional (Fase 2)
   - Setup específico da story → dentro da fase dessa story

### Estrutura de Fases

- **Fase 1**: Setup (inicialização do projeto)
- **Fase 2**: Fundacional (pré-requisitos bloqueantes — DEVE completar antes das user stories)
- **Fase 3+**: User Stories em ordem de prioridade (P1, P2, P3...)
  - Dentro de cada story: Testes (se solicitados) → Models → Services → Endpoints → Integração
  - Cada fase deve ser um incremento completo e testável independentemente
- **Fase Final**: Polimento e Preocupações Transversais

## Concluído Quando

- [ ] tasks.md gerado com todas as fases, IDs de tarefa e caminhos de arquivo
- [ ] Hooks de extensão despachados ou ignorados conforme as regras em Hooks Obrigatórios Pós-Execução acima
- [ ] Conclusão reportada ao usuário com contagem de tarefas, breakdown por story e escopo MVP
