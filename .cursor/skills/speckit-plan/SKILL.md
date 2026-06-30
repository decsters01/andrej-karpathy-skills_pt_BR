---
name: "speckit-plan"
description: "Executar o fluxo de trabalho de planejamento de implementação usando o template de plano para gerar artefatos de design."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/plan.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes do planejamento)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_plan`
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

1. **Configuração**: Execute `.specify/scripts/bash/setup-plan.sh --json` a partir da raiz do repositório e analise o JSON para FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

2. **Carregar contexto**: Leia FEATURE_SPEC e `.specify/memory/constitution.md`. Carregue o template IMPL_PLAN (já copiado).

3. **Executar fluxo de plano**: Siga a estrutura no template IMPL_PLAN para:
   - Preencher Contexto Técnico (marque desconhecidos como "NEEDS CLARIFICATION")
   - Preencher seção Constitution Check a partir da constitution
   - Avaliar gates (ERROR se violações não justificadas)
   - Fase 0: Gerar research.md (resolver todos os NEEDS CLARIFICATION)
   - Fase 1: Gerar data-model.md, contracts/, quickstart.md
   - Fase 1: Atualizar contexto do agente executando o script do agente
   - Reavaliar Constitution Check pós-design

## Hooks Obrigatórios Pós-Execução

**Você DEVE completar esta seção antes de reportar a conclusão ao usuário.**

Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se não existir, ou nenhum hook estiver registrado sob `hooks.after_plan`, pule para o Relatório de Conclusão.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_plan`.
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

O comando termina após o planejamento da Fase 2. Reporte branch, caminho IMPL_PLAN e artefatos gerados.

## Fases

### Fase 0: Roteiro e Pesquisa

1. **Extrair desconhecidos do Contexto Técnico** acima:
   - Para cada NEEDS CLARIFICATION → tarefa de pesquisa
   - Para cada dependência → tarefa de melhores práticas
   - Para cada integração → tarefa de padrões

2. **Gerar e despachar agentes de pesquisa**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidar descobertas** em `research.md` usando o formato:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Saída**: research.md com todos os NEEDS CLARIFICATION resolvidos

### Fase 1: Design e Contratos

**Pré-requisitos:** `research.md` completo

1. **Extrair entidades da especificação da feature** → `data-model.md`:
   - Nome da entidade, campos, relacionamentos
   - Regras de validação dos requisitos
   - Transições de estado, se aplicável

2. **Definir contratos de interface** (se o projeto tiver interfaces externas) → `/contracts/`:
   - Identificar quais interfaces o projeto expõe a usuários ou outros sistemas
   - Documentar o formato de contrato apropriado para o tipo de projeto
   - Exemplos: APIs públicas para bibliotecas, schemas de comando para ferramentas CLI, endpoints para serviços web, gramáticas para parsers, contratos de UI para aplicações
   - Pule se o projeto for puramente interno (scripts de build, ferramentas pontuais, etc.)

3. **Criar guia de validação quickstart** → `quickstart.md`:
   - Documentar cenários de validação executáveis que comprovem que a feature funciona ponta a ponta
   - Incluir pré-requisitos, comandos de setup, comandos de teste/execução e resultados esperados
   - Usar links ou referências a contratos e detalhes do modelo de dados em vez de duplicá-los
   - Não incluir código de implementação completo, corpos de model/service/controller, migrações ou suítes de teste completas
   - Manter este artefato como guia de validação/execução; detalhes de implementação pertencem a `tasks.md` e à fase de implementação

**Saída**: data-model.md, /contracts/*, quickstart.md

## Regras principais

- Use caminhos absolutos para operações no filesystem; use caminhos relativos ao projeto para referências na documentação
- ERROR em falhas de gate ou esclarecimentos não resolvidos

## Concluído Quando

- [ ] Fluxo de plano executado e artefatos de design gerados
- [ ] Hooks de extensão despachados ou ignorados conforme as regras em Hooks Obrigatórios Pós-Execução acima
- [ ] Conclusão reportada ao usuário com branch, caminho do plano e artefatos gerados
