---
name: "speckit-analyze"
description: "Realizar análise não destrutiva de consistência e qualidade entre spec.md, plan.md e tasks.md após a geração de tarefas."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/analyze.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da análise)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_analyze`
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

    Aguarde o resultado do comando do hook antes de prosseguir para o Objetivo.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Objetivo

Identificar inconsistências, duplicações, ambiguidades e itens subespecificados entre os três artefatos principais (`spec.md`, `plan.md`, `tasks.md`) antes da implementação. Este comando DEVE executar apenas após `/speckit-tasks` ter produzido com sucesso um `tasks.md` completo.

## Restrições Operacionais

**ESTRITAMENTE SOMENTE LEITURA**: **Não** modifique nenhum arquivo. Produza um relatório de análise estruturado. Ofereça um plano de remediação opcional (o usuário deve aprovar explicitamente antes que quaisquer comandos de edição de acompanhamento sejam invocados manualmente).

**Autoridade da Constitution**: A constitution do projeto (`.specify/memory/constitution.md`) é **não negociável** neste escopo de análise. Conflitos com a constitution são automaticamente CRÍTICOS e exigem ajuste da spec, plan ou tasks — não diluição, reinterpretação ou ignorar silenciosamente o princípio. Se um princípio em si precisar mudar, isso deve ocorrer em uma atualização explícita separada da constitution fora de `/speckit-analyze`.

## Etapas de Execução

### 1. Inicializar Contexto de Análise

Execute `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` uma vez a partir da raiz do repositório e analise o JSON para FEATURE_DIR e AVAILABLE_DOCS. Derive caminhos absolutos:

- SPEC = FEATURE_DIR/spec.md
- PLAN = FEATURE_DIR/plan.md
- TASKS = FEATURE_DIR/tasks.md

Interrompa com mensagem de erro se qualquer arquivo obrigatório estiver ausente (instrua o usuário a executar o comando pré-requisito ausente).
Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

### 2. Carregar Artefatos (Divulgação Progressiva)

Carregue apenas o contexto mínimo necessário de cada artefato:

**De spec.md:**

- Visão Geral/Contexto
- Requisitos Funcionais
- Critérios de Sucesso (resultados mensuráveis — ex. performance, segurança, disponibilidade, sucesso do usuário, impacto de negócio)
- User Stories
- Casos Extremos (se presentes)

**De plan.md:**

- Escolhas de arquitetura/stack
- Referências do Modelo de Dados
- Fases
- Restrições técnicas

**De tasks.md:**

- IDs de tarefa
- Descrições
- Agrupamento por fase
- Marcadores paralelos [P]
- Caminhos de arquivo referenciados

**Da constitution:**

- Carregue `.specify/memory/constitution.md` para validação de princípios

### 3. Construir Modelos Semânticos

Crie representações internas (não inclua artefatos brutos na saída):

- **Inventário de requisitos**: Para cada Requisito Funcional (FR-###) e Critério de Sucesso (SC-###), registre uma chave estável. Use o identificador explícito FR-/SC- como chave primária quando presente, e opcionalmente derive um slug de frase imperativa para legibilidade (ex. "Usuário pode fazer upload de arquivo" → `user-can-upload-file`). Inclua apenas itens de Critérios de Sucesso que exijam trabalho construível (ex. infraestrutura de teste de carga, ferramentas de auditoria de segurança), e exclua métricas de resultado pós-lançamento e KPIs de negócio (ex. "Reduzir tickets de suporte em 50%").
- **Inventário de user story/ação**: Ações discretas do usuário com critérios de aceitação
- **Mapeamento de cobertura de tarefas**: Mapeie cada tarefa para um ou mais requisitos ou stories (inferência por palavra-chave / padrões de referência explícita como IDs ou frases-chave)
- **Conjunto de regras da constitution**: Extraia nomes de princípios e declarações normativas MUST/SHOULD

### 4. Passes de Detecção (Análise Eficiente em Tokens)

Foque em descobertas de alto sinal. Limite a 50 descobertas no total; agregue o restante em resumo de excedente.

#### A. Detecção de Duplicação

- Identifique requisitos quase duplicados
- Marque formulações de menor qualidade para consolidação

#### B. Detecção de Ambiguidade

- Sinalize adjetivos vagos (fast, scalable, secure, intuitive, robust) sem critérios mensuráveis
- Sinalize placeholders não resolvidos (TODO, TKTK, ???, `<placeholder>`, etc.)

#### C. Subespecificação

- Requisitos com verbos mas sem objeto ou resultado mensurável
- User stories sem alinhamento de critérios de aceitação
- Tarefas referenciando arquivos ou componentes não definidos em spec/plan

#### D. Alinhamento com Constitution

- Qualquer requisito ou elemento de plano conflitando com um princípio MUST
- Seções ou quality gates obrigatórios ausentes da constitution

#### E. Lacunas de Cobertura

- Requisitos com zero tarefas associadas
- Tarefas sem requisito/story mapeado
- Critérios de Sucesso exigindo trabalho construível (performance, segurança, disponibilidade) não refletidos em tarefas

#### F. Inconsistência

- Deriva de terminologia (mesmo conceito nomeado diferentemente entre arquivos)
- Entidades de dados referenciadas no plan mas ausentes na spec (ou vice-versa)
- Contradições de ordenação de tarefas (ex. tarefas de integração antes de tarefas de setup fundacional sem nota de dependência)
- Requisitos conflitantes (ex. um exige Next.js enquanto outro especifica Vue)

### 5. Atribuição de Severidade

Use esta heurística para priorizar descobertas:

- **CRÍTICO**: Viola MUST da constitution, artefato principal da spec ausente, ou requisito com cobertura zero que bloqueia funcionalidade base
- **ALTA**: Requisito duplicado ou conflitante, atributo de segurança/performance ambíguo, critério de aceitação não testável
- **MÉDIA**: Deriva de terminologia, cobertura de tarefa não funcional ausente, caso extremo subespecificado
- **BAIXA**: Melhorias de estilo/redação, redundância menor que não afeta ordem de execução

### 6. Produzir Relatório de Análise Compacto

Produza um relatório Markdown (sem escrita em arquivo) com a seguinte estrutura:

## Relatório de Análise da Especificação

| ID | Categoria | Severidade | Local(is) | Resumo | Recomendação |
|----|----------|----------|-------------|---------|----------------|
| A1 | Duplicação | ALTA | spec.md:L120-134 | Dois requisitos similares ... | Mesclar formulação; manter versão mais clara |

(Adicione uma linha por descoberta; gere IDs estáveis prefixados pela inicial da categoria.)

**Tabela de Resumo de Cobertura:**

| Chave do Requisito | Tem Tarefa? | IDs de Tarefa | Notas |
|-----------------|-----------|----------|-------|

**Problemas de Alinhamento com Constitution:** (se houver)

**Tarefas Não Mapeadas:** (se houver)

**Métricas:**

- Total de Requisitos
- Total de Tarefas
- % de Cobertura (requisitos com >=1 tarefa)
- Contagem de Ambiguidades
- Contagem de Duplicações
- Contagem de Problemas Críticos

### 7. Fornecer Próximas Ações

No final do relatório, produza um bloco conciso de Próximas Ações:

- Se existirem problemas CRÍTICOS: Recomende resolver antes de `/speckit-implement`
- Se apenas BAIXA/MÉDIA: Usuário pode prosseguir, mas forneça sugestões de melhoria
- Forneça sugestões explícitas de comando: ex. "Execute /speckit-specify com refinamento", "Execute /speckit-plan para ajustar arquitetura", "Edite manualmente tasks.md para adicionar cobertura para 'métricas-de-performance'"

### 8. Oferecer Remediação

Pergunte ao usuário: "Gostaria que eu sugerisse edições concretas de remediação para os principais N problemas?" (NÃO as aplique automaticamente.)

### 9. Verificar hooks de extensão

Após reportar, verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_analyze`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) com hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
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

## Princípios Operacionais

### Eficiência de Contexto

- **Tokens mínimos de alto sinal**: Foque em descobertas acionáveis, não documentação exaustiva
- **Divulgação progressiva**: Carregue artefatos incrementalmente; não despeje todo o conteúdo na análise
- **Saída eficiente em tokens**: Limite tabela de descobertas a 50 linhas; resuma excedente
- **Resultados determinísticos**: Reexecutar sem mudanças deve produzir IDs e contagens consistentes

### Diretrizes de Análise

- **NUNCA modifique arquivos** (esta é análise somente leitura)
- **NUNCA alucine seções ausentes** (se ausentes, reporte com precisão)
- **Priorize violações da constitution** (estas são sempre CRÍTICAS)
- **Use exemplos em vez de regras exaustivas** (cite instâncias específicas, não padrões genéricos)
- **Reporte zero problemas graciosamente** (emita relatório de sucesso com estatísticas de cobertura)

## Contexto

$ARGUMENTS
