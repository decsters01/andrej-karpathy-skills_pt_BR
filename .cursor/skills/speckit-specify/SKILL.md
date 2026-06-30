---
name: "speckit-specify"
description: "Criar ou atualizar a especificação da feature a partir de uma descrição de feature em linguagem natural."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/specify.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da especificação)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_specify`
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

O texto que o usuário digitou após `/speckit-specify` na mensagem de disparo **é** a descrição da feature. Assuma que você sempre o tem disponível nesta conversa mesmo se `$ARGUMENTS` aparecer literalmente abaixo. Não peça ao usuário para repetir a menos que ele tenha fornecido um comando vazio.

Dada essa descrição da feature, faça o seguinte:

1. **Gere um nome curto conciso** (2-4 palavras) para a feature:
   - Analise a descrição da feature e extraia as palavras-chave mais significativas
   - Crie um nome curto de 2-4 palavras que capture a essência da feature
   - Use formato ação-substantivo quando possível (ex. "add-user-auth", "fix-payment-bug")
   - Preserve termos técnicos e siglas (OAuth2, API, JWT, etc.)
   - Mantenha conciso mas descritivo o suficiente para entender a feature de relance
   - Exemplos:
     - "Quero adicionar autenticação de usuário" → "user-auth"
     - "Implementar integração OAuth2 para a API" → "oauth2-api-integration"
     - "Criar um dashboard para analytics" → "analytics-dashboard"
     - "Corrigir bug de timeout no processamento de pagamento" → "fix-payment-timeout"

2. **Criação de branch** (opcional, via hook):

   Se um hook `before_specify` executou com sucesso nas Verificações Pré-Execução acima, ele terá criado/alternado para uma branch git e produzido JSON contendo `BRANCH_NAME` e `FEATURE_NUM`. Anote esses valores para referência, mas o nome da branch **não** dita o nome do diretório da spec.

   Se o usuário forneceu explicitamente `GIT_BRANCH_NAME`, repasse-o ao hook para que o script de branch use o valor exato como nome da branch (ignorando toda geração de prefixo/sufixo).

3. **Criar o diretório da feature de spec**:

   Specs ficam sob o diretório padrão `specs/` a menos que o usuário forneça explicitamente `SPECIFY_FEATURE_DIRECTORY`.

   **Ordem de resolução para `SPECIFY_FEATURE_DIRECTORY`**:
   1. Se o usuário forneceu explicitamente `SPECIFY_FEATURE_DIRECTORY` (ex. via variável de ambiente, argumento ou configuração), use-o como está
   2. Caso contrário, gere automaticamente sob `specs/`:
      - Verifique `.specify/init-options.json` para `feature_numbering` (preferido) ou `branch_numbering` (obsoleto, apenas migração — será removido em versão futura)
      - Se `"timestamp"`: prefixo é `YYYYMMDD-HHMMSS` (timestamp atual)
      - Se `"sequential"` ou ausente: prefixo é `NNN` (próximo número de 3 dígitos disponível após escanear diretórios existentes em `specs/`)
      - Construa o nome do diretório: `<prefix>-<short-name>` (ex. `003-user-auth` ou `20260319-143022-user-auth`)
      - Defina `SPECIFY_FEATURE_DIRECTORY` como `specs/<directory-name>`
      - Se `branch_numbering` foi usado (e `feature_numbering` estava ausente), emita um aviso de uma linha: "⚠️ `branch_numbering` em init-options.json está obsoleto. Renomeie para `feature_numbering`."

   **Criar o diretório e arquivo de spec**:
   - `mkdir -p SPECIFY_FEATURE_DIRECTORY`
   - Resolva o `spec-template` ativo através da pilha de resolução de preset/template do Spec Kit (equivalente a `specify preset resolve spec-template`)
   - Copie o arquivo `spec-template` resolvido para `SPECIFY_FEATURE_DIRECTORY/spec.md` como ponto de partida
   - Defina `SPEC_FILE` como `SPECIFY_FEATURE_DIRECTORY/spec.md`
   - Persista o caminho resolvido em `.specify/feature.json`:
     ```json
     {
       "feature_directory": "<resolved feature dir>"
     }
     ```
     Escreva o valor real do caminho do diretório resolvido (por exemplo, `specs/003-user-auth`), não a string literal `SPECIFY_FEATURE_DIRECTORY`.
     Isso permite que comandos downstream (`/speckit-plan`, `/speckit-tasks`, etc.) localizem o diretório da feature sem depender de convenções de nome de branch git.

   **IMPORTANTE**:
   - Você deve criar apenas uma feature por invocação de `/speckit-specify`
   - O nome do diretório da spec e o nome da branch git são independentes — podem ser iguais, mas isso é escolha do usuário
   - O diretório e arquivo da spec são sempre criados por este comando, nunca pelo hook

4. Carregue o arquivo `spec-template` ativo resolvido para entender as seções obrigatórias.

5. **SE EXISTIR**: Carregue `.specify/memory/constitution.md` para princípios do projeto e restrições de governança.

6. Siga este fluxo de execução:
    1. Analise a descrição do usuário dos argumentos
       Se vazia: ERRO "Nenhuma descrição de feature fornecida"
    2. Extraia conceitos-chave da descrição
       Identifique: atores, ações, dados, restrições
    3. Para aspectos pouco claros:
       - Faça suposições informadas com base no contexto e padrões da indústria
       - Marque apenas com [PRECISA DE ESCLARECIMENTO: specific question] se:
         - A escolha impacta significativamente o escopo da feature ou experiência do usuário
         - Múltiplas interpretações razoáveis existem com implicações diferentes
         - Nenhum default razoável existe
       - **LIMITE: Máximo de 3 marcadores [PRECISA DE ESCLARECIMENTO] no total**
       - Priorize esclarecimentos por impacto: escopo > segurança/privacidade > experiência do usuário > detalhes técnicos
    4. Preencha a seção Cenários de Usuário e Testes
       Se não houver fluxo de usuário claro: ERRO "Não é possível determinar cenários de usuário"
    5. Gere Requisitos Funcionais
       Cada requisito deve ser testável
       Use defaults razoáveis para detalhes não especificados (documente suposições na seção Premissas)
    6. Defina Critérios de Sucesso
       Crie resultados mensuráveis e agnósticos de tecnologia
       Inclua métricas quantitativas (tempo, performance, volume) e medidas qualitativas (satisfação do usuário, conclusão de tarefa)
       Cada critério deve ser verificável sem detalhes de implementação
    7. Identifique Entidades Principais (se envolver dados)
    8. Retorne: SUCCESS (spec pronta para planejamento)

6. Escreva a especificação em SPEC_FILE usando a estrutura do template, substituindo placeholders por detalhes concretos derivados da descrição da feature (argumentos) preservando ordem de seções e headings.

7. **Validação de Qualidade da Especificação**: Após escrever a spec inicial, valide-a contra critérios de qualidade:

   a. **Criar Checklist de Qualidade da Spec**: Gere um arquivo de checklist em `SPECIFY_FEATURE_DIRECTORY/checklists/requirements.md` usando a estrutura do template de checklist com estes itens de validação:

      ```markdown
      # Checklist de Qualidade da Especificação: [NOME DA FEATURE]
      
      **Propósito**: Validar completude e qualidade da especificação antes de prosseguir para o planejamento
      **Criado**: [DATA]
      **Feature**: [Link para spec.md]
      
      ## Qualidade do Conteúdo
      
      - [ ] Sem detalhes de implementação (linguagens, frameworks, APIs)
      - [ ] Focado em valor para o usuário e necessidades de negócio
      - [ ] Escrito para stakeholders não técnicos
      - [ ] Todas as seções obrigatórias completadas
      
      ## Completude dos Requisitos
      
      - [ ] Nenhum marcador [PRECISA DE ESCLARECIMENTO] permanece
      - [ ] Requisitos são testáveis e inequívocos
      - [ ] Critérios de sucesso são mensuráveis
      - [ ] Critérios de sucesso são agnósticos de tecnologia (sem detalhes de implementação)
      - [ ] Todos os cenários de aceitação estão definidos
      - [ ] Casos extremos estão identificados
      - [ ] Escopo está claramente delimitado
      - [ ] Dependências e premissas identificadas
      
      ## Prontidão da Feature
      
      - [ ] Todos os requisitos funcionais têm critérios de aceitação claros
      - [ ] Cenários de usuário cobrem fluxos principais
      - [ ] Feature atende resultados mensuráveis definidos nos Critérios de Sucesso
      - [ ] Nenhum detalhe de implementação vaza para a especificação
      
      ## Notas
      
      - Itens marcados como incompletos exigem atualizações na spec antes de `/speckit-clarify` ou `/speckit-plan`
      ```

   b. **Executar Verificação de Validação**: Revise a spec contra cada item da checklist:
      - Para cada item, determine se passa ou falha
      - Documente problemas específicos encontrados (cite seções relevantes da spec)

   c. **Tratar Resultados da Validação**:

      - **Se todos os itens passarem**: Marque a checklist como completa e prossiga para a seção Hooks Obrigatórios Pós-Execução

      - **Se itens falharem (excluindo [PRECISA DE ESCLARECIMENTO])**:
        1. Liste os itens que falharam e problemas específicos
        2. Atualize a spec para resolver cada problema
        3. Reexecute a validação até todos os itens passarem (máx. 3 iterações)
        4. Se ainda falhar após 3 iterações, documente problemas restantes nas notas da checklist e avise o usuário

      - **Se marcadores [PRECISA DE ESCLARECIMENTO] permanecerem**:
        1. Extraia todos os marcadores [PRECISA DE ESCLARECIMENTO: ...] da spec
        2. **VERIFICAÇÃO DE LIMITE**: Se existirem mais de 3 marcadores, mantenha apenas os 3 mais críticos (por impacto escopo/segurança/UX) e faça suposições informadas para o restante
        3. Para cada esclarecimento necessário (máx. 3), apresente opções ao usuário neste formato:

           ```markdown
           ## Pergunta [N]: [Tópico]
           
           **Contexto**: [Citar seção relevante da spec]
           
           **O que precisamos saber**: [Pergunta específica do marcador PRECISA DE ESCLARECIMENTO]
           
           **Respostas Sugeridas**:
           
           | Opção | Resposta | Implicações |
           |--------|--------|--------------|
           | A      | [Primeira resposta sugerida] | [O que isso significa para a feature] |
           | B      | [Segunda resposta sugerida] | [O que isso significa para a feature] |
           | C      | [Terceira resposta sugerida] | [O que isso significa para a feature] |
           | Personalizado | Forneça sua própria resposta | [Explique como fornecer entrada personalizada] |
           
           **Sua escolha**: _[Aguarde resposta do usuário]_
           ```

        4. **CRÍTICO - Formatação de Tabela**: Garanta que tabelas markdown estejam formatadas corretamente:
           - Use espaçamento consistente com pipes alinhados
           - Cada célula deve ter espaços ao redor do conteúdo: `| Conteúdo |` não `|Conteúdo|`
           - Separador de cabeçalho deve ter pelo menos 3 traços: `|--------|`
           - Teste que a tabela renderiza corretamente na pré-visualização markdown
        5. Numere perguntas sequencialmente (Q1, Q2, Q3 - máx. 3 no total)
        6. Apresente todas as perguntas juntas antes de aguardar respostas
        7. Aguarde o usuário responder com suas escolhas para todas as perguntas (ex. "Q1: A, Q2: Personalizado - [detalhes], Q3: B")
        8. Atualize a spec substituindo cada marcador [PRECISA DE ESCLARECIMENTO] pela resposta selecionada ou fornecida pelo usuário
        9. Reexecute a validação após todos os esclarecimentos serem resolvidos

   d. **Atualizar Checklist**: Após cada iteração de validação, atualize o arquivo de checklist com o status atual de aprovado/reprovado

## Hooks Obrigatórios Pós-Execução

**Você DEVE completar esta seção antes de reportar a conclusão ao usuário.**

Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se não existir, ou nenhum hook estiver registrado sob `hooks.after_specify`, pule para o Relatório de Conclusão.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_specify`.
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue para o Relatório de Conclusão.
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook obrigatório** (`optional: false`) — **Você DEVE emitir `EXECUTE_COMMAND:` para cada hook obrigatório**:
    ```
    ## Hooks de Extensão

    **Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
  - **Hook opcional** (`optional: true`):
    ```
    ## Hooks de Extensão

    **Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```

## Relatório de Conclusão

Reporte a conclusão ao usuário com:
- `SPECIFY_FEATURE_DIRECTORY` — o caminho do diretório da feature
- `SPEC_FILE` — o caminho do arquivo de spec
- Resumo dos resultados da checklist
- Prontidão para a próxima fase (`/speckit-clarify` ou `/speckit-plan`)

**NOTA:** A criação de branch é tratada pelo hook `before_specify` (extensão git). Criação de diretório e arquivo de spec são sempre tratadas por este comando core.

## Diretrizes Rápidas

- Foque no **O QUÊ** os usuários precisam e **POR QUÊ**.
- Evite COMO implementar (sem stack técnica, APIs, estrutura de código).
- Escrito para stakeholders de negócio, não desenvolvedores.
- NÃO crie checklists embutidas na spec. Isso será um comando separado.

### Requisitos de Seção

- **Seções obrigatórias**: Devem ser completadas para cada feature
- **Seções opcionais**: Inclua apenas quando relevante para a feature
- Quando uma seção não se aplicar, remova-a inteiramente (não deixe como "N/A")

### Para Geração por IA

Ao criar esta spec a partir de um prompt do usuário:

1. **Faça suposições informadas**: Use contexto, padrões da indústria e padrões comuns para preencher lacunas
2. **Documente premissas**: Registre defaults razoáveis na seção Premissas
3. **Limite esclarecimentos**: Máximo de 3 marcadores [PRECISA DE ESCLARECIMENTO] — use apenas para decisões críticas que:
   - Impactam significativamente o escopo da feature ou experiência do usuário
   - Têm múltiplas interpretações razoáveis com implicações diferentes
   - Não têm nenhum default razoável
4. **Priorize esclarecimentos**: escopo > segurança/privacidade > experiência do usuário > detalhes técnicos
5. **Pense como um testador**: Cada requisito vago deve falhar no item de checklist "testável e inequívoco"
6. **Áreas comuns que precisam de esclarecimento** (apenas se nenhum default razoável existir):
   - Escopo e limites da feature (incluir/excluir casos de uso específicos)
   - Tipos de usuário e permissões (se múltiplas interpretações conflitantes forem possíveis)
   - Requisitos de segurança/compliance (quando legal/financeiramente significativos)

**Exemplos de defaults razoáveis** (não pergunte sobre estes):

- Retenção de dados: Práticas padrão da indústria para o domínio
- Metas de performance: Expectativas padrão de apps web/mobile a menos que especificado
- Tratamento de erros: Mensagens amigáveis com fallbacks apropriados
- Método de autenticação: Sessão padrão ou OAuth2 para apps web
- Padrões de integração: Use padrões apropriados ao projeto (REST/GraphQL para serviços web, chamadas de função para bibliotecas, args CLI para ferramentas, etc.)

### Diretrizes de Critérios de Sucesso

Critérios de sucesso devem ser:

1. **Mensuráveis**: Inclua métricas específicas (tempo, porcentagem, contagem, taxa)
2. **Agnósticos de tecnologia**: Sem menção a frameworks, linguagens, bancos de dados ou ferramentas
3. **Focados no usuário**: Descreva resultados da perspectiva do usuário/negócio, não internos do sistema
4. **Verificáveis**: Podem ser testados/validados sem conhecer detalhes de implementação

**Bons exemplos**:

- "Usuários conseguem concluir o checkout em menos de 3 minutos"
- "O sistema suporta 10.000 usuários concorrentes"
- "95% das buscas retornam resultados em menos de 1 segundo"
- "Taxa de conclusão de tarefas melhora em 40%"

**Maus exemplos** (focados em implementação):

- "Tempo de resposta da API abaixo de 200ms" (muito técnico, use "Usuários veem resultados instantaneamente")
- "Banco de dados consegue lidar com 1000 TPS" (detalhe de implementação, use métrica voltada ao usuário)
- "Componentes React renderizam eficientemente" (específico de framework)
- "Taxa de acerto do cache Redis acima de 80%" (específico de tecnologia)

## Concluído Quando

- [ ] Especificação escrita em `SPEC_FILE` e validada contra checklist de qualidade
- [ ] Hooks de extensão despachados ou ignorados conforme as regras em Hooks Obrigatórios Pós-Execução acima
- [ ] Conclusão reportada ao usuário com diretório da feature, caminho do arquivo de spec e resultados da checklist
