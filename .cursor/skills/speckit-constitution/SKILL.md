---
name: "speckit-constitution"
description: "Criar ou atualizar a constitution do projeto a partir de entradas de princípios interativas ou fornecidas, garantindo que todos os templates dependentes permaneçam sincronizados."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/constitution.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da atualização da constitution)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_constitution`
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

Você está atualizando a constitution do projeto em `.specify/memory/constitution.md`. Este arquivo é um TEMPLATE contendo tokens placeholder entre colchetes (ex. `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`). Seu trabalho é (a) coletar/derivar valores concretos, (b) preencher o template com precisão e (c) propagar quaisquer emendas pelos artefatos dependentes.

**Nota**: Se `.specify/memory/constitution.md` ainda não existir, ele deveria ter sido inicializado a partir de `.specify/templates/constitution-template.md` durante o setup do projeto. Se estiver ausente, copie o template primeiro.

Siga este fluxo de execução:

1. Carregue a constitution existente em `.specify/memory/constitution.md`.
   - Identifique cada token placeholder da forma `[ALL_CAPS_IDENTIFIER]`.
   **IMPORTANTE**: O usuário pode exigir menos ou mais princípios do que os usados no template. Se um número for especificado, respeite — siga o template geral. Você atualizará o documento de acordo.

2. Colete/derive valores para placeholders:
   - Se a entrada do usuário (conversa) fornecer um valor, use-o.
   - Caso contrário, infira do contexto existente do repositório (README, docs, versões anteriores da constitution se embutidas).
   - Para datas de governança: `RATIFICATION_DATE` é a data de adoção original (se desconhecida, pergunte ou marque TODO), `LAST_AMENDED_DATE` é hoje se houver alterações, caso contrário mantenha a anterior.
   - `CONSTITUTION_VERSION` deve incrementar conforme regras de versionamento semântico:
     - MAJOR: Remoções ou redefinições incompatíveis de governança/princípios.
     - MINOR: Novo princípio/seção adicionado ou orientação materialmente expandida.
     - PATCH: Esclarecimentos, ajustes de redação, correções de typo, refinamentos não semânticos.
   - Se o tipo de bump de versão for ambíguo, proponha o raciocínio antes de finalizar.

3. Redija o conteúdo atualizado da constitution:
   - Substitua cada placeholder por texto concreto (sem tokens entre colchetes restantes, exceto slots de template intencionalmente retidos que o projeto escolheu não definir ainda — justifique explicitamente qualquer um deixado).
   - Preserve a hierarquia de headings e comentários podem ser removidos após substituição, a menos que ainda agreguem orientação esclarecedora.
   - Garanta que cada seção de Princípio: linha de nome sucinta, parágrafo (ou lista com bullets) capturando regras não negociáveis, justificativa explícita se não for óbvia.
   - Garanta que a seção Governança liste procedimento de emenda, política de versionamento e expectativas de revisão de conformidade.

4. Checklist de propagação de consistência (converta checklist anterior em validações ativas):
   - Leia `.specify/templates/plan-template.md` e garanta que qualquer "Constitution Check" ou regras estejam alinhados com os princípios atualizados.
   - Leia `.specify/templates/spec-template.md` para alinhamento de escopo/requisitos — atualize se a constitution adicionar/remover seções ou restrições obrigatórias.
   - Leia `.specify/templates/tasks-template.md` e garanta que a categorização de tarefas reflita tipos de tarefa orientados por princípios novos ou removidos (ex. observabilidade, versionamento, disciplina de testes).
   - Leia cada arquivo de comando em `.specify/templates/commands/*.md` (incluindo este) para verificar que não restem referências desatualizadas (nomes específicos de agente como CLAUDE apenas) quando orientação genérica for necessária.
   - Leia quaisquer docs de orientação em runtime (ex. `README.md`, `docs/quickstart.md`, ou arquivos de orientação específicos de agente se presentes). Atualize referências a princípios alterados.

5. Produza um Relatório de Impacto de Sincronização (prependa como comentário HTML no topo do arquivo da constitution após atualização):
   - Mudança de versão: antiga → nova
   - Lista de princípios modificados (título antigo → novo título se renomeado)
   - Seções adicionadas
   - Seções removidas
   - Templates que requerem atualizações (✅ atualizado / ⚠ pendente) com caminhos de arquivo
   - TODOs de acompanhamento se algum placeholder foi intencionalmente adiado.

6. Validação antes da saída final:
   - Sem tokens entre colchetes restantes não explicados.
   - Linha de versão corresponde ao relatório.
   - Datas em formato ISO YYYY-MM-DD.
   - Princípios são declarativos, testáveis e livres de linguagem vaga ("should" → substitua por MUST/SHOULD com justificativa quando apropriado).

7. Escreva a constitution completa de volta em `.specify/memory/constitution.md` (sobrescreva).

8. Produza um resumo final ao usuário com:
   - Nova versão e justificativa do bump.
   - Quaisquer arquivos sinalizados para acompanhamento manual.
   - Mensagem de commit sugerida (ex. `docs: amend constitution to vX.Y.Z (principle additions + governance update)`).

Requisitos de Formatação e Estilo:

- Use headings Markdown exatamente como no template (não rebaixe/promova níveis).
- Quebre linhas longas de justificativa para manter legibilidade (<100 chars idealmente), mas não force quebras estranhas.
- Mantenha uma linha em branco entre seções.
- Evite espaços em branco no final das linhas.

Se o usuário fornecer atualizações parciais (ex. apenas uma revisão de princípio), ainda execute as etapas de validação e decisão de versão.

Se informação crítica estiver ausente (ex. data de ratificação verdadeiramente desconhecida), insira `TODO(<FIELD_NAME>): explanation` e inclua no Relatório de Impacto de Sincronização sob itens adiados.

Não crie um novo template; sempre opere no arquivo `.specify/memory/constitution.md` existente.

## Verificações Pós-Execução

**Verificar hooks de extensão (após atualização da constitution)**:
Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_constitution`
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

    **Hook Automático**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente
