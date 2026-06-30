---
name: "speckit-checklist"
description: "Gerar uma checklist personalizada para a feature atual com base nos requisitos do usuário."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/checklist.md"
---


## Propósito da Checklist: "Unit Tests for English"

**CONCEITO CRÍTICO**: Checklists são **TESTES UNITÁRIOS PARA ESCRITA DE REQUISITOS** — elas validam a qualidade, clareza e completude dos requisitos em um determinado domínio.

**NÃO para verificação/teste**:

- ❌ NÃO "Verificar se o botão clica corretamente"
- ❌ NÃO "Testar se o tratamento de erros funciona"
- ❌ NÃO "Confirmar se a API retorna 200"
- ❌ NÃO verificar se código/implementação corresponde à spec

**PARA validação da qualidade dos requisitos**:

- ✅ "Os requisitos de hierarquia visual estão definidos para todos os tipos de card?" (completude)
- ✅ "'Exibição proeminente' está quantificada com tamanho/posicionamento específicos?" (clareza)
- ✅ "Os requisitos de estado hover são consistentes em todos os elementos interativos?" (consistência)
- ✅ "Os requisitos de acessibilidade estão definidos para navegação por teclado?" (cobertura)
- ✅ "A spec define o que acontece quando a imagem do logo falha ao carregar?" (casos extremos)

**Metáfora**: Se sua spec é código escrito em inglês, a checklist é sua suíte de testes unitários. Você está testando se os requisitos estão bem escritos, completos, inequívocos e prontos para implementação — NÃO se a implementação funciona.

## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da geração de checklist)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_checklist`
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

    Aguarde o resultado do comando do hook antes de prosseguir para as Etapas de Execução.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Etapas de Execução

1. **Configuração**: Execute `.specify/scripts/bash/check-prerequisites.sh --json` a partir da raiz do repositório e analise o JSON para FEATURE_DIR e a lista AVAILABLE_DOCS.
   - Todos os caminhos de arquivo devem ser absolutos.
   - Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

2. **SE EXISTIR**: Carregue `.specify/memory/constitution.md` para princípios do projeto e restrições de governança.

3. **Esclarecer intenção (dinâmico)**: Derive até TRÊS perguntas contextuais iniciais de esclarecimento (sem catálogo pré-definido). Elas DEVEM:
   - Ser geradas da formulação do usuário + sinais extraídos de spec/plan/tasks
   - Perguntar apenas sobre informação que muda materialmente o conteúdo da checklist
   - Ser puladas individualmente se já estiverem inequívocas em `$ARGUMENTS`
   - Preferir precisão em vez de amplitude

   Algoritmo de geração:
   1. Extraia sinais: palavras-chave do domínio da feature (ex. auth, latency, UX, API), indicadores de risco ("critical", "must", "compliance"), dicas de stakeholders ("QA", "review", "security team") e entregáveis explícitos ("a11y", "rollback", "contracts").
   2. Agrupe sinais em áreas de foco candidatas (máx. 4) ranqueadas por relevância.
   3. Identifique audiência e timing prováveis (autor, revisor, QA, release) se não explícitos.
   4. Detecte dimensões ausentes: amplitude de escopo, profundidade/rigor, ênfase de risco, limites de exclusão, critérios de aceitação mensuráveis.
   5. Formule perguntas escolhidas destes arquétipos:
      - Refinamento de escopo (ex. "Deve incluir pontos de integração com X e Y ou permanecer limitado à correção do módulo local?")
      - Priorização de risco (ex. "Quais destas áreas de risco potenciais devem receber verificações obrigatórias de gate?")
      - Calibração de profundidade (ex. "É uma lista de sanidade leve pré-commit ou um gate formal de release?")
      - Enquadramento de audiência (ex. "Será usada apenas pelo autor ou por pares durante revisão de PR?")
      - Exclusão de limites (ex. "Devemos excluir explicitamente itens de ajuste de performance nesta rodada?")
      - Lacuna de classe de cenário (ex. "Nenhum fluxo de recuperação detectado — caminhos de rollback / falha parcial estão no escopo?")

   Regras de formatação de perguntas:
   - Se apresentar opções, gere uma tabela compacta com colunas: Option | Candidate | Why It Matters
   - Limite a opções A–E no máximo; omita tabela se resposta livre for mais clara
   - Nunca peça ao usuário para repetir o que já disse
   - Evite categorias especulativas (sem alucinação). Se incerto, pergunte explicitamente: "Confirm whether X belongs in scope."

   Defaults quando interação for impossível:
   - Profundidade: Standard
   - Audiência: Reviewer (PR) se relacionado a código; Author caso contrário
   - Foco: Top 2 clusters de relevância

   Produza as perguntas (rotule Q1/Q2/Q3). Após respostas: se ≥2 classes de cenário (Alternate / Exception / Recovery / Non-Functional domain) permanecerem pouco claras, você PODE fazer até DUAS perguntas de acompanhamento direcionadas (Q4/Q5) com justificativa de uma linha cada (ex. "Unresolved recovery path risk"). Não exceda cinco perguntas no total. Pule escalonamento se o usuário recusar explicitamente mais perguntas.

4. **Entender solicitação do usuário**: Combine `$ARGUMENTS` + respostas de esclarecimento:
   - Derive tema da checklist (ex. security, review, deploy, ux)
   - Consolide itens obrigatórios explícitos mencionados pelo usuário
   - Mapeie seleções de foco para estrutura de categorias
   - Infira qualquer contexto ausente de spec/plan/tasks (NÃO alucine)

5. **Carregar contexto da feature**: Leia de FEATURE_DIR:
   - spec.md: Requisitos e escopo da feature
   - plan.md (se existir): Detalhes técnicos, dependências
   - tasks.md (se existir): Tarefas de implementação

   **Estratégia de Carregamento de Contexto**:
   - Carregue apenas porções necessárias relevantes às áreas de foco ativas (evite dump completo de arquivo)
   - Prefira resumir seções longas em bullets concisos de cenário/requisito
   - Use divulgação progressiva: adicione recuperação adicional apenas se lacunas forem detectadas
   - Se docs fonte forem grandes, gere itens de resumo intermediários em vez de embutir texto bruto

6. **Gerar checklist** - Crie "Unit Tests for Requirements":
   - Crie o diretório `FEATURE_DIR/checklists/` se não existir
   - Gere nome de arquivo de checklist único:
     - Use nome curto e descritivo baseado no domínio (ex. `ux.md`, `api.md`, `security.md`)
     - Formato: `[domain].md`
   - Comportamento de manipulação de arquivo:
     - Se o arquivo NÃO existir: Crie novo arquivo e numere itens começando em CHK001
     - Se o arquivo existir: Acrescente novos itens ao arquivo existente, continuando do último ID CHK (ex. se o último item é CHK015, comece novos itens em CHK016)
   - Nunca delete ou substitua conteúdo existente da checklist — sempre preserve e acrescente

   **PRINCÍPIO CENTRAL - Teste os Requisitos, Não a Implementação**:
   Cada item de checklist DEVE avaliar os PRÓPRIOS REQUISITOS quanto a:
   - **Completude**: Todos os requisitos necessários estão presentes?
   - **Clareza**: Os requisitos são inequívocos e específicos?
   - **Consistência**: Os requisitos se alinham entre si?
   - **Mensurabilidade**: Os requisitos podem ser verificados objetivamente?
   - **Cobertura**: Todos os cenários/casos extremos estão endereçados?

   **Estrutura de Categorias** - Agrupe itens por dimensões de qualidade de requisitos:
   - **Requirement Completeness** (Todos os requisitos necessários estão documentados?)
   - **Requirement Clarity** (Os requisitos são específicos e inequívocos?)
   - **Requirement Consistency** (Os requisitos se alinham sem conflitos?)
   - **Acceptance Criteria Quality** (Os critérios de sucesso são mensuráveis?)
   - **Scenario Coverage** (Todos os fluxos/casos estão endereçados?)
   - **Edge Case Coverage** (Condições de limite estão definidas?)
   - **Non-Functional Requirements** (Performance, Security, Accessibility, etc. — estão especificados?)
   - **Dependencies & Assumptions** (Estão documentados e validados?)
   - **Ambiguities & Conflicts** (O que precisa de esclarecimento?)

   **COMO ESCREVER ITENS DE CHECKLIST - "Unit Tests for English"**:

   ❌ **ERRADO** (Testando implementação):
   - "Verify landing page displays 3 episode cards"
   - "Test hover states work on desktop"
   - "Confirm logo click navigates home"

   ✅ **CORRETO** (Testando qualidade dos requisitos):
   - "Are the exact number and layout of featured episodes specified?" [Completeness]
   - "Is 'prominent display' quantified with specific sizing/positioning?" [Clarity]
   - "Are hover state requirements consistent across all interactive elements?" [Consistency]
   - "Are keyboard navigation requirements defined for all interactive UI?" [Coverage]
   - "Is the fallback behavior specified when logo image fails to load?" [Edge Cases]
   - "Are loading states defined for asynchronous episode data?" [Completeness]
   - "Does the spec define visual hierarchy for competing UI elements?" [Clarity]

   **ESTRUTURA DO ITEM**:
   Cada item deve seguir este padrão:
   - Formato de pergunta sobre qualidade do requisito
   - Foque no que está ESCRITO (ou não escrito) na spec/plan
   - Inclua dimensão de qualidade entre colchetes [Completeness/Clarity/Consistency/etc.]
   - Referencie seção da spec `[Spec §X.Y]` ao verificar requisitos existentes
   - Use marcador `[Gap]` ao verificar requisitos ausentes

   **EXEMPLOS POR DIMENSÃO DE QUALIDADE**:

   Completeness:
   - "Are error handling requirements defined for all API failure modes? [Gap]"
   - "Are accessibility requirements specified for all interactive elements? [Completeness]"
   - "Are mobile breakpoint requirements defined for responsive layouts? [Gap]"

   Clarity:
   - "Is 'fast loading' quantified with specific timing thresholds? [Clarity, Spec §NFR-2]"
   - "Are 'related episodes' selection criteria explicitly defined? [Clarity, Spec §FR-5]"
   - "Is 'prominent' defined with measurable visual properties? [Ambiguity, Spec §FR-4]"

   Consistency:
   - "Do navigation requirements align across all pages? [Consistency, Spec §FR-10]"
   - "Are card component requirements consistent between landing and detail pages? [Consistency]"

   Coverage:
   - "Are requirements defined for zero-state scenarios (no episodes)? [Coverage, Edge Case]"
   - "Are concurrent user interaction scenarios addressed? [Coverage, Gap]"
   - "Are requirements specified for partial data loading failures? [Coverage, Exception Flow]"

   Measurability:
   - "Are visual hierarchy requirements measurable/testable? [Acceptance Criteria, Spec §FR-1]"
   - "Can 'balanced visual weight' be objectively verified? [Measurability, Spec §FR-2]"

   **Classificação e Cobertura de Cenários** (Foco em Qualidade de Requisitos):
   - Verifique se existem requisitos para: Primary, Alternate, Exception/Error, Recovery, Non-Functional scenarios
   - Para cada classe de cenário, pergunte: "Are [scenario type] requirements complete, clear, and consistent?"
   - Se classe de cenário ausente: "Are [scenario type] requirements intentionally excluded or missing? [Gap]"
   - Inclua resiliência/rollback quando ocorrer mutação de estado: "Are rollback requirements defined for migration failures? [Gap]"

   **Requisitos de Rastreabilidade**:
   - MÍNIMO: ≥80% dos itens DEVEM incluir pelo menos uma referência de rastreabilidade
   - Cada item deve referenciar: seção da spec `[Spec §X.Y]`, ou usar marcadores: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`
   - Se não existir sistema de IDs: "Is a requirement & acceptance criteria ID scheme established? [Traceability]"

   **Expor e Resolver Problemas** (Problemas de Qualidade de Requisitos):
   Faça perguntas sobre os próprios requisitos:
   - Ambiguidades: "Is the term 'fast' quantified with specific metrics? [Ambiguity, Spec §NFR-1]"
   - Conflitos: "Do navigation requirements conflict between §FR-10 and §FR-10a? [Conflict]"
   - Premissas: "Is the assumption of 'always available podcast API' validated? [Assumption]"
   - Dependências: "Are external podcast API requirements documented? [Dependency, Gap]"
   - Definições ausentes: "Is 'visual hierarchy' defined with measurable criteria? [Gap]"

   **Consolidação de Conteúdo**:
   - Limite suave: Se itens candidatos brutos > 40, priorize por risco/impacto
   - Mescle quase-duplicatas verificando o mesmo aspecto de requisito
   - Se >5 casos extremos de baixo impacto, crie um item: "Are edge cases X, Y, Z addressed in requirements? [Coverage]"

   **🚫 ABSOLUTAMENTE PROIBIDO** - Estes tornam um teste de implementação, não de requisitos:
   - ❌ Qualquer item começando com "Verify", "Test", "Confirm", "Check" + comportamento de implementação
   - ❌ Referências a execução de código, ações do usuário, comportamento do sistema
   - ❌ "Displays correctly", "works properly", "functions as expected"
   - ❌ "Click", "navigate", "render", "load", "execute"
   - ❌ Casos de teste, planos de teste, procedimentos de QA
   - ❌ Detalhes de implementação (frameworks, APIs, algoritmos)

   **✅ PADRÕES OBRIGATÓRIOS** - Estes testam qualidade dos requisitos:
   - ✅ "Are [requirement type] defined/specified/documented for [scenario]?"
   - ✅ "Is [vague term] quantified/clarified with specific criteria?"
   - ✅ "Are requirements consistent between [section A] and [section B]?"
   - ✅ "Can [requirement] be objectively measured/verified?"
   - ✅ "Are [edge cases/scenarios] addressed in requirements?"
   - ✅ "Does the spec define [missing aspect]?"

7. **Referência de Estrutura**: Gere a checklist seguindo o template canônico em `.specify/templates/checklist-template.md` para título, seção meta, headings de categoria e formatação de ID. Se o template estiver indisponível, use: título H1, linhas meta de propósito/criado, seções de categoria `##` contendo linhas `- [ ] CHK### <requirement item>` com IDs globalmente incrementais começando em CHK001.

8. **Relatório**: Produza o caminho completo do arquivo de checklist, contagem de itens e resuma se a execução criou um novo arquivo ou acrescentou a um existente. Resuma:
   - Áreas de foco selecionadas
   - Nível de profundidade
   - Ator/timing
   - Quaisquer itens obrigatórios explícitos especificados pelo usuário incorporados

**Importante**: Cada invocação do comando `/speckit-checklist` usa um nome de arquivo de checklist curto e descritivo e ou cria um novo arquivo ou acrescenta a um existente. Isso permite:

- Múltiplas checklists de tipos diferentes (ex. `ux.md`, `test.md`, `security.md`)
- Nomes de arquivo simples e memoráveis que indicam o propósito da checklist
- Identificação e navegação fáceis na pasta `checklists/`

Para evitar desordem, use tipos descritivos e limpe checklists obsoletas quando terminar.

## Tipos de Checklist de Exemplo e Itens de Amostra

**Qualidade de Requisitos UX:** `ux.md`

Itens de amostra (testando os requisitos, NÃO a implementação):

- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]"
- "Are interaction state requirements (hover, focus, active) consistently defined? [Consistency]"
- "Are accessibility requirements specified for all interactive elements? [Coverage, Gap]"
- "Is fallback behavior defined when images fail to load? [Edge Case, Gap]"
- "Can 'prominent display' be objectively measured? [Measurability, Spec §FR-4]"

**Qualidade de Requisitos API:** `api.md`

Itens de amostra:

- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"
- "Are authentication requirements consistent across all endpoints? [Consistency]"
- "Are retry/timeout requirements defined for external dependencies? [Coverage, Gap]"
- "Is versioning strategy documented in requirements? [Gap]"

**Qualidade de Requisitos de Performance:** `performance.md`

Itens de amostra:

- "Are performance requirements quantified with specific metrics? [Clarity]"
- "Are performance targets defined for all critical user journeys? [Coverage]"
- "Are performance requirements under different load conditions specified? [Completeness]"
- "Can performance requirements be objectively measured? [Measurability]"
- "Are degradation requirements defined for high-load scenarios? [Edge Case, Gap]"

**Qualidade de Requisitos de Segurança:** `security.md`

Itens de amostra:

- "Are authentication requirements specified for all protected resources? [Coverage]"
- "Are data protection requirements defined for sensitive information? [Completeness]"
- "Is the threat model documented and requirements aligned to it? [Traceability]"
- "Are security requirements consistent with compliance obligations? [Consistency]"
- "Are security failure/breach response requirements defined? [Gap, Exception Flow]"

## Anti-Exemplos: O Que NÃO Fazer

**❌ ERRADO - Estes testam implementação, não requisitos:**

```markdown
- [ ] CHK001 - Verify landing page displays 3 episode cards [Spec §FR-001]
- [ ] CHK002 - Test hover states work correctly on desktop [Spec §FR-003]
- [ ] CHK003 - Confirm logo click navigates to home page [Spec §FR-010]
- [ ] CHK004 - Check that related episodes section shows 3-5 items [Spec §FR-005]
```

**✅ CORRETO - Estes testam qualidade dos requisitos:**

```markdown
- [ ] CHK001 - Are the number and layout of featured episodes explicitly specified? [Completeness, Spec §FR-001]
- [ ] CHK002 - Are hover state requirements consistently defined for all interactive elements? [Consistency, Spec §FR-003]
- [ ] CHK003 - Are navigation requirements clear for all clickable brand elements? [Clarity, Spec §FR-010]
- [ ] CHK004 - Is the selection criteria for related episodes documented? [Gap, Spec §FR-005]
- [ ] CHK005 - Are loading state requirements defined for asynchronous episode data? [Gap]
- [ ] CHK006 - Can "visual hierarchy" requirements be objectively measured? [Measurability, Spec §FR-001]
```

**Diferenças Principais:**

- Errado: Testa se o sistema funciona corretamente
- Correto: Testa se os requisitos estão escritos corretamente
- Errado: Verificação de comportamento
- Correto: Validação da qualidade dos requisitos
- Errado: "Does it do X?"
- Correto: "Is X clearly specified?"

## Verificações Pós-Execução

**Verificar hooks de extensão (após geração de checklist)**:
Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_checklist`
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

    **Hook Opcional**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    Para executar: `/{command}`
    ```
  - **Hook obrigatório** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente
