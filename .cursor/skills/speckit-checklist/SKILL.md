---
name: "speckit-checklist"
description: "Gerar uma checklist personalizada para a feature atual com base nos requisitos do usuário."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/checklist.md"
---


## Propósito da Checklist: "Testes Unitários para Escrita de Requisitos"

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

**Metáfora**: Se sua spec é o "código-fonte" dos requisitos, a checklist é sua suíte de testes unitários. Você está testando se os requisitos estão bem escritos, completos, inequívocos e prontos para implementação — NÃO se a implementação funciona.

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
   - Se apresentar opções, gere uma tabela compacta com colunas: Opção | Candidato | Por Que Importa
   - Limite a opções A–E no máximo; omita tabela se resposta livre for mais clara
   - Nunca peça ao usuário para repetir o que já disse
   - Evite categorias especulativas (sem alucinação). Se incerto, pergunte explicitamente: "Confirme se X pertence ao escopo."

   Padrões quando interação for impossível:
   - Profundidade: Padrão
   - Audiência: Revisor (PR) se relacionado a código; Autor caso contrário
   - Foco: 2 principais clusters de relevância

   Produza as perguntas (rotule Q1/Q2/Q3). Após respostas: se ≥2 classes de cenário (Alternativo / Exceção / Recuperação / domínio Não Funcional) permanecerem pouco claras, você PODE fazer até DUAS perguntas de acompanhamento direcionadas (Q4/Q5) com justificativa de uma linha cada (ex. "Risco de caminho de recuperação não resolvido"). Não exceda cinco perguntas no total. Pule escalonamento se o usuário recusar explicitamente mais perguntas.

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

6. **Gerar checklist** - Crie "Testes Unitários para Requisitos":
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
   - **Completude dos Requisitos** (Todos os requisitos necessários estão documentados?)
   - **Clareza dos Requisitos** (Os requisitos são específicos e inequívocos?)
   - **Consistência dos Requisitos** (Os requisitos se alinham sem conflitos?)
   - **Qualidade dos Critérios de Aceitação** (Os critérios de sucesso são mensuráveis?)
   - **Cobertura de Cenários** (Todos os fluxos/casos estão endereçados?)
   - **Cobertura de Casos Extremos** (Condições de limite estão definidas?)
   - **Requisitos Não Funcionais** (Performance, Segurança, Acessibilidade, etc. — estão especificados?)
   - **Dependências e Premissas** (Estão documentados e validados?)
   - **Ambiguidades e Conflitos** (O que precisa de esclarecimento?)

   **COMO ESCREVER ITENS DE CHECKLIST - "Testes Unitários para Escrita de Requisitos"**:

   ❌ **ERRADO** (Testando implementação):
   - "Verificar se a landing page exibe 3 cards de episódio"
   - "Testar se estados hover funcionam no desktop"
   - "Confirmar se clique no logo navega para home"

   ✅ **CORRETO** (Testando qualidade dos requisitos):
   - "O número exato e o layout dos episódios em destaque estão especificados?" [Completude]
   - "'Exibição proeminente' está quantificada com tamanho/posicionamento específicos?" [Clareza]
   - "Os requisitos de estado hover são consistentes em todos os elementos interativos?" [Consistência]
   - "Os requisitos de navegação por teclado estão definidos para toda a UI interativa?" [Cobertura]
   - "O comportamento de fallback está especificado quando a imagem do logo falha ao carregar?" [Casos Extremos]
   - "Os estados de carregamento estão definidos para dados assíncronos de episódios?" [Completude]
   - "A spec define hierarquia visual para elementos de UI concorrentes?" [Clareza]

   **ESTRUTURA DO ITEM**:
   Cada item deve seguir este padrão:
   - Formato de pergunta sobre qualidade do requisito
   - Foque no que está ESCRITO (ou não escrito) na spec/plan
   - Inclua dimensão de qualidade entre colchetes [Completude/Clareza/Consistência/etc.]
   - Referencie seção da spec `[Spec §X.Y]` ao verificar requisitos existentes
   - Use marcador `[Lacuna]` ao verificar requisitos ausentes

   **EXEMPLOS POR DIMENSÃO DE QUALIDADE**:

   Completude:
   - "Os requisitos de tratamento de erros estão definidos para todos os modos de falha da API? [Lacuna]"
   - "Os requisitos de acessibilidade estão especificados para todos os elementos interativos? [Completude]"
   - "Os requisitos de breakpoint mobile estão definidos para layouts responsivos? [Lacuna]"

   Clareza:
   - "'Carregamento rápido' está quantificado com limites de tempo específicos? [Clareza, Spec §NFR-2]"
   - "Os critérios de seleção de 'episódios relacionados' estão explicitamente definidos? [Clareza, Spec §FR-5]"
   - "'Proeminente' está definido com propriedades visuais mensuráveis? [Ambiguidade, Spec §FR-4]"

   Consistência:
   - "Os requisitos de navegação se alinham em todas as páginas? [Consistência, Spec §FR-10]"
   - "Os requisitos do componente card são consistentes entre landing e páginas de detalhe? [Consistência]"

   Cobertura:
   - "Os requisitos estão definidos para cenários de estado zero (sem episódios)? [Cobertura, Caso Extremo]"
   - "Os cenários de interação concorrente de usuários estão endereçados? [Cobertura, Lacuna]"
   - "Os requisitos estão especificados para falhas parciais de carregamento de dados? [Cobertura, Fluxo de Exceção]"

   Mensurabilidade:
   - "Os requisitos de hierarquia visual são mensuráveis/testáveis? [Critérios de Aceitação, Spec §FR-1]"
   - "'Peso visual equilibrado' pode ser verificado objetivamente? [Mensurabilidade, Spec §FR-2]"

   **Classificação e Cobertura de Cenários** (Foco em Qualidade de Requisitos):
   - Verifique se existem requisitos para: Primário, Alternativo, Exceção/Erro, Recuperação, cenários Não Funcionais
   - Para cada classe de cenário, pergunte: "Os requisitos de [tipo de cenário] estão completos, claros e consistentes?"
   - Se classe de cenário ausente: "Os requisitos de [tipo de cenário] foram intencionalmente excluídos ou estão ausentes? [Lacuna]"
   - Inclua resiliência/rollback quando ocorrer mutação de estado: "Os requisitos de rollback estão definidos para falhas de migração? [Lacuna]"

   **Requisitos de Rastreabilidade**:
   - MÍNIMO: ≥80% dos itens DEVEM incluir pelo menos uma referência de rastreabilidade
   - Cada item deve referenciar: seção da spec `[Spec §X.Y]`, ou usar marcadores: `[Lacuna]`, `[Ambiguidade]`, `[Conflito]`, `[Premissa]`
   - Se não existir sistema de IDs: "Um esquema de IDs de requisitos e critérios de aceitação está estabelecido? [Rastreabilidade]"

   **Expor e Resolver Problemas** (Problemas de Qualidade de Requisitos):
   Faça perguntas sobre os próprios requisitos:
   - Ambiguidades: "O termo 'rápido' está quantificado com métricas específicas? [Ambiguidade, Spec §NFR-1]"
   - Conflitos: "Os requisitos de navegação conflitam entre §FR-10 e §FR-10a? [Conflito]"
   - Premissas: "A premissa de 'API de podcast sempre disponível' está validada? [Premissa]"
   - Dependências: "Os requisitos da API externa de podcast estão documentados? [Dependência, Lacuna]"
   - Definições ausentes: "'Hierarquia visual' está definida com critérios mensuráveis? [Lacuna]"

   **Consolidação de Conteúdo**:
   - Limite suave: Se itens candidatos brutos > 40, priorize por risco/impacto
   - Mescle quase-duplicatas verificando o mesmo aspecto de requisito
   - Se >5 casos extremos de baixo impacto, crie um item: "Os casos extremos X, Y, Z estão endereçados nos requisitos? [Cobertura]"

   **🚫 ABSOLUTAMENTE PROIBIDO** - Estes tornam um teste de implementação, não de requisitos:
   - ❌ Qualquer item começando com "Verificar", "Testar", "Confirmar", "Checar" + comportamento de implementação
   - ❌ Referências a execução de código, ações do usuário, comportamento do sistema
   - ❌ "Exibe corretamente", "funciona adequadamente", "comporta-se como esperado"
   - ❌ "Clicar", "navegar", "renderizar", "carregar", "executar"
   - ❌ Casos de teste, planos de teste, procedimentos de QA
   - ❌ Detalhes de implementação (frameworks, APIs, algoritmos)

   **✅ PADRÕES OBRIGATÓRIOS** - Estes testam qualidade dos requisitos:
   - ✅ "Os [tipo de requisito] estão definidos/especificados/documentados para [cenário]?"
   - ✅ "[termo vago] está quantificado/esclarecido com critérios específicos?"
   - ✅ "Os requisitos são consistentes entre [seção A] e [seção B]?"
   - ✅ "[requisito] pode ser objetivamente medido/verificado?"
   - ✅ "[casos extremos/cenários] estão endereçados nos requisitos?"
   - ✅ "A spec define [aspecto ausente]?"

7. **Referência de Estrutura**: Gere a checklist seguindo o template canônico em `.specify/templates/checklist-template.md` para título, seção meta, headings de categoria e formatação de ID. Se o template estiver indisponível, use: título H1, linhas meta de propósito/criado, seções de categoria `##` contendo linhas `- [ ] CHK### <item de requisito>` com IDs globalmente incrementais começando em CHK001.

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

- "Os requisitos de hierarquia visual estão definidos com critérios mensuráveis? [Clareza, Spec §FR-1]"
- "O número e posicionamento dos elementos de UI estão explicitamente especificados? [Completude, Spec §FR-1]"
- "Os requisitos de estado de interação (hover, focus, active) estão consistentemente definidos? [Consistência]"
- "Os requisitos de acessibilidade estão especificados para todos os elementos interativos? [Cobertura, Lacuna]"
- "O comportamento de fallback está definido quando imagens falham ao carregar? [Caso Extremo, Lacuna]"
- "'Exibição proeminente' pode ser objetivamente medida? [Mensurabilidade, Spec §FR-4]"

**Qualidade de Requisitos API:** `api.md`

Itens de amostra:

- "Os formatos de resposta de erro estão especificados para todos os cenários de falha? [Completude]"
- "Os requisitos de rate limiting estão quantificados com limites específicos? [Clareza]"
- "Os requisitos de autenticação são consistentes em todos os endpoints? [Consistência]"
- "Os requisitos de retry/timeout estão definidos para dependências externas? [Cobertura, Lacuna]"
- "A estratégia de versionamento está documentada nos requisitos? [Lacuna]"

**Qualidade de Requisitos de Performance:** `performance.md`

Itens de amostra:

- "Os requisitos de performance estão quantificados com métricas específicas? [Clareza]"
- "As metas de performance estão definidas para todas as jornadas críticas do usuário? [Cobertura]"
- "Os requisitos de performance sob diferentes condições de carga estão especificados? [Completude]"
- "Os requisitos de performance podem ser objetivamente medidos? [Mensurabilidade]"
- "Os requisitos de degradação estão definidos para cenários de alta carga? [Caso Extremo, Lacuna]"

**Qualidade de Requisitos de Segurança:** `security.md`

Itens de amostra:

- "Os requisitos de autenticação estão especificados para todos os recursos protegidos? [Cobertura]"
- "Os requisitos de proteção de dados estão definidos para informações sensíveis? [Completude]"
- "O modelo de ameaça está documentado e os requisitos alinhados a ele? [Rastreabilidade]"
- "Os requisitos de segurança são consistentes com obrigações de compliance? [Consistência]"
- "Os requisitos de resposta a falha/violação de segurança estão definidos? [Lacuna, Fluxo de Exceção]"

## Anti-Exemplos: O Que NÃO Fazer

**❌ ERRADO - Estes testam implementação, não requisitos:**

```markdown
- [ ] CHK001 - Verificar se a landing page exibe 3 cards de episódio [Spec §FR-001]
- [ ] CHK002 - Testar se estados hover funcionam corretamente no desktop [Spec §FR-003]
- [ ] CHK003 - Confirmar se clique no logo navega para a home [Spec §FR-010]
- [ ] CHK004 - Checar se a seção de episódios relacionados mostra 3-5 itens [Spec §FR-005]
```

**✅ CORRETO - Estes testam qualidade dos requisitos:**

```markdown
- [ ] CHK001 - O número e layout dos episódios em destaque estão explicitamente especificados? [Completude, Spec §FR-001]
- [ ] CHK002 - Os requisitos de estado hover estão consistentemente definidos para todos os elementos interativos? [Consistência, Spec §FR-003]
- [ ] CHK003 - Os requisitos de navegação estão claros para todos os elementos de marca clicáveis? [Clareza, Spec §FR-010]
- [ ] CHK004 - Os critérios de seleção de episódios relacionados estão documentados? [Lacuna, Spec §FR-005]
- [ ] CHK005 - Os requisitos de estado de carregamento estão definidos para dados assíncronos de episódios? [Lacuna]
- [ ] CHK006 - Os requisitos de "hierarquia visual" podem ser objetivamente medidos? [Mensurabilidade, Spec §FR-001]
```

**Diferenças Principais:**

- Errado: Testa se o sistema funciona corretamente
- Correto: Testa se os requisitos estão escritos corretamente
- Errado: Verificação de comportamento
- Correto: Validação da qualidade dos requisitos
- Errado: "Ele faz X?"
- Correto: "X está claramente especificado?"

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
