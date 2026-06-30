---
name: "speckit-implement"
description: "Executar o plano de implementação processando e executando todas as tarefas definidas em tasks.md"
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/implement.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes da implementação)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_implement`
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

1. Execute `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` a partir da raiz do repositório e analise FEATURE_DIR e a lista AVAILABLE_DOCS. Todos os caminhos devem ser absolutos. Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

2. **Verificar status das checklists** (se FEATURE_DIR/checklists/ existir):
   - Escaneie todos os arquivos de checklist no diretório checklists/
   - Para cada checklist, conte:
     - Total de itens: Todas as linhas correspondendo a `- [ ]` ou `- [X]` ou `- [x]`
     - Itens concluídos: Linhas correspondendo a `- [X]` ou `- [x]`
     - Itens incompletos: Linhas correspondendo a `- [ ]`
   - Crie uma tabela de status:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - Calcule o status geral:
     - **PASS**: Todas as checklists têm 0 itens incompletos
     - **FAIL**: Uma ou mais checklists têm itens incompletos

   - **Se alguma checklist estiver incompleta**:
     - Exiba a tabela com contagens de itens incompletos
     - **PARE** e pergunte: "Algumas checklists estão incompletas. Deseja prosseguir com a implementação mesmo assim? (yes/no)"
     - Aguarde resposta do usuário antes de continuar
     - Se o usuário disser "no" ou "wait" ou "stop", interrompa a execução
     - Se o usuário disser "yes" ou "proceed" ou "continue", prossiga para a etapa 3

   - **Se todas as checklists estiverem completas**:
     - Exiba a tabela mostrando que todas as checklists passaram
     - Prossiga automaticamente para a etapa 3

3. Carregue e analise o contexto de implementação:
   - **OBRIGATÓRIO**: Leia tasks.md para a lista completa de tarefas e plano de execução
   - **OBRIGATÓRIO**: Leia plan.md para stack técnica, arquitetura e estrutura de arquivos
   - **SE EXISTIR**: Leia data-model.md para entidades e relacionamentos
   - **SE EXISTIR**: Leia contracts/ para especificações de API e requisitos de teste
   - **SE EXISTIR**: Leia research.md para decisões técnicas e restrições
   - **SE EXISTIR**: Leia .specify/memory/constitution.md para restrições de governança
   - **SE EXISTIR**: Leia quickstart.md para cenários de integração

4. **Verificação de Setup do Projeto**:
   - **OBRIGATÓRIO**: Crie/verifique arquivos ignore com base no setup real do projeto:

   **Lógica de Detecção e Criação**:
   - Verifique se o seguinte comando tem sucesso para determinar se o repositório é um repo git (crie/verifique .gitignore se for):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Verifique se Dockerfile* existe ou Docker em plan.md → crie/verifique .dockerignore
   - Verifique se .eslintrc* existe → crie/verifique .eslintignore
   - Verifique se eslint.config.* existe → garanta que as entradas `ignores` da config cubram os padrões necessários
   - Verifique se .prettierrc* existe → crie/verifique .prettierignore
   - Verifique se .npmrc ou package.json existe → crie/verifique .npmignore (se publicando)
   - Verifique se arquivos terraform (*.tf) existem → crie/verifique .terraformignore
   - Verifique se .helmignore é necessário (helm charts presentes) → crie/verifique .helmignore

   **Se o arquivo ignore já existir**: Verifique se contém padrões essenciais, acrescente apenas padrões críticos ausentes
   **Se o arquivo ignore estiver ausente**: Crie com conjunto completo de padrões para a tecnologia detectada

   **Padrões Comuns por Tecnologia** (da stack técnica de plan.md):
   - **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `*.dll`, `autom4te.cache/`, `config.status`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **Padrões Específicos de Ferramentas**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. Analise a estrutura de tasks.md e extraia:
   - **Fases de tarefa**: Setup, Tests, Core, Integration, Polish
   - **Dependências de tarefa**: Regras de execução sequencial vs paralela
   - **Detalhes de tarefa**: ID, descrição, caminhos de arquivo, marcadores paralelos [P]
   - **Fluxo de execução**: Ordem e requisitos de dependência

6. Execute a implementação seguindo o plano de tarefas:
   - **Execução fase por fase**: Complete cada fase antes de passar para a próxima
   - **Respeite dependências**: Execute tarefas sequenciais em ordem, tarefas paralelas [P] podem executar juntas  
   - **Siga abordagem TDD**: Execute tarefas de teste antes das tarefas de implementação correspondentes
   - **Coordenação baseada em arquivo**: Tarefas que afetam os mesmos arquivos devem executar sequencialmente
   - **Checkpoints de validação**: Verifique conclusão de cada fase antes de prosseguir

7. Regras de execução de implementação:
   - **Setup primeiro**: Inicialize estrutura do projeto, dependências, configuração
   - **Testes antes do código**: Se precisar escrever testes para contratos, entidades e cenários de integração
   - **Desenvolvimento core**: Implemente models, services, comandos CLI, endpoints
   - **Trabalho de integração**: Conexões de banco de dados, middleware, logging, serviços externos
   - **Polimento e validação**: Testes unitários, otimização de performance, documentação

8. Rastreamento de progresso e tratamento de erros:
   - Reporte progresso após cada tarefa concluída
   - Interrompa execução se qualquer tarefa não paralela falhar
   - Para tarefas paralelas [P], continue com as bem-sucedidas, reporte as que falharam
   - Forneça mensagens de erro claras com contexto para depuração
   - Sugira próximos passos se a implementação não puder prosseguir
   - **IMPORTANTE** Para tarefas concluídas, certifique-se de marcar a tarefa como [X] no arquivo de tarefas.

9. Validação de conclusão:
   - Verifique se todas as tarefas obrigatórias estão concluídas
   - Verifique se as features implementadas correspondem à especificação original
   - Valide que os testes passam e a cobertura atende aos requisitos
   - Confirme que a implementação segue o plano técnico

Nota: Este comando assume que existe um breakdown completo de tarefas em tasks.md. Se as tarefas estiverem incompletas ou ausentes, sugira executar `/speckit-tasks` primeiro para regenerar a lista de tarefas.

## Hooks Obrigatórios Pós-Execução

**Você DEVE completar esta seção antes de reportar a conclusão ao usuário.**

Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se não existir, ou nenhum hook estiver registrado sob `hooks.after_implement`, pule para o Relatório de Conclusão.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_implement`.
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

Reporte status final com resumo do trabalho concluído.

## Concluído Quando

- [ ] Todas as tarefas em tasks.md concluídas e marcadas `[X]`
- [ ] Implementação validada contra especificação, plano e cobertura de testes
- [ ] Hooks de extensão despachados ou ignorados conforme as regras em Hooks Obrigatórios Pós-Execução acima
- [ ] Conclusão reportada ao usuário com resumo do trabalho concluído
