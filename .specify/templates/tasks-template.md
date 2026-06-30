---

description: "Template de lista de tarefas para implementação de feature"
---

# Tarefas: [FEATURE NAME]

**Entrada**: Documentos de design de `/specs/[###-feature-name]/`

**Pré-requisitos**: plan.md (obrigatório), spec.md (obrigatório para user stories), research.md, data-model.md, contracts/

**Testes**: Os exemplos abaixo incluem tarefas de teste. Testes são OPCIONAIS — inclua-os apenas se explicitamente solicitado na especificação da feature.

**Organização**: Tarefas são agrupadas por user story para permitir implementação e teste independentes de cada story.

## Formato: `[ID] [P?] [Story] Description`

- **[P]**: Pode executar em paralelo (arquivos diferentes, sem dependências)
- **[Story]**: A qual user story esta tarefa pertence (ex. US1, US2, US3)
- Inclua caminhos exatos de arquivo nas descrições

## Convenções de Caminho

- **Projeto único**: `src/`, `tests/` na raiz do repositório
- **Web app**: `backend/src/`, `frontend/src/`
- **Mobile**: `api/src/`, `ios/src/` ou `android/src/`
- Caminhos mostrados abaixo assumem projeto único — ajuste com base na estrutura de plan.md

<!--
  ============================================================================
  IMPORTANTE: As tarefas abaixo são TAREFAS DE EXEMPLO apenas para fins de ilustração.

  O comando /speckit-tasks DEVE substituí-las por tarefas reais com base em:
  - User stories de spec.md (com suas prioridades P1, P2, P3...)
  - Requisitos da feature de plan.md
  - Entidades de data-model.md
  - Endpoints de contracts/

  Tarefas DEVEM ser organizadas por user story para que cada story possa ser:
  - Implementada independentemente
  - Testada independentemente
  - Entregue como incremento MVP

  NÃO mantenha estas tarefas de exemplo no arquivo tasks.md gerado.
  ============================================================================
-->

## Fase 1: Setup (Infraestrutura Compartilhada)

**Propósito**: Inicialização do projeto e estrutura básica

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize [language] project with [framework] dependencies
- [ ] T003 [P] Configure linting and formatting tools

---

## Fase 2: Fundacional (Pré-requisitos Bloqueantes)

**Propósito**: Infraestrutura core que DEVE estar completa antes de QUALQUER user story poder ser implementada

**⚠️ CRÍTICO**: Nenhum trabalho de user story pode começar até esta fase estar completa

Exemplos de tarefas fundacionais (ajuste com base no seu projeto):

- [ ] T004 Setup database schema and migrations framework
- [ ] T005 [P] Implement authentication/authorization framework
- [ ] T006 [P] Setup API routing and middleware structure
- [ ] T007 Create base models/entities that all stories depend on
- [ ] T008 Configure error handling and logging infrastructure
- [ ] T009 Setup environment configuration management

**Checkpoint**: Fundação pronta — implementação de user story pode começar em paralelo agora

---

## Fase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Objetivo**: [Brief description of what this story delivers]

**Teste Independente**: [How to verify this story works on its own]

### Testes para User Story 1 (OPCIONAL - apenas se testes solicitados) ⚠️

> **NOTA: Escreva estes testes PRIMEIRO, garanta que FALHEM antes da implementação**

- [ ] T010 [P] [US1] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T011 [P] [US1] Integration test for [user journey] in tests/integration/test_[name].py

### Implementação para User Story 1

- [ ] T012 [P] [US1] Create [Entity1] model in src/models/[entity1].py
- [ ] T013 [P] [US1] Create [Entity2] model in src/models/[entity2].py
- [ ] T014 [US1] Implement [Service] in src/services/[service].py (depends on T012, T013)
- [ ] T015 [US1] Implement [endpoint/feature] in src/[location]/[file].py
- [ ] T016 [US1] Add validation and error handling
- [ ] T017 [US1] Add logging for user story 1 operations

**Checkpoint**: Neste ponto, User Story 1 deve estar totalmente funcional e testável independentemente

---

## Fase 4: User Story 2 - [Title] (Priority: P2)

**Objetivo**: [Brief description of what this story delivers]

**Teste Independente**: [How to verify this story works on its own]

### Testes para User Story 2 (OPCIONAL - apenas se testes solicitados) ⚠️

- [ ] T018 [P] [US2] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T019 [P] [US2] Integration test for [user journey] in tests/integration/test_[name].py

### Implementação para User Story 2

- [ ] T020 [P] [US2] Create [Entity] model in src/models/[entity].py
- [ ] T021 [US2] Implement [Service] in src/services/[service].py
- [ ] T022 [US2] Implement [endpoint/feature] in src/[location]/[file].py
- [ ] T023 [US2] Integrate with User Story 1 components (if needed)

**Checkpoint**: Neste ponto, User Stories 1 E 2 devem funcionar independentemente

---

## Fase 5: User Story 3 - [Title] (Priority: P3)

**Objetivo**: [Brief description of what this story delivers]

**Teste Independente**: [How to verify this story works on its own]

### Testes para User Story 3 (OPCIONAL - apenas se testes solicitados) ⚠️

- [ ] T024 [P] [US3] Contract test for [endpoint] in tests/contract/test_[name].py
- [ ] T025 [P] [US3] Integration test for [user journey] in tests/integration/test_[name].py

### Implementação para User Story 3

- [ ] T026 [P] [US3] Create [Entity] model in src/models/[entity].py
- [ ] T027 [US3] Implement [Service] in src/services/[service].py
- [ ] T028 [US3] Implement [endpoint/feature] in src/[location]/[file].py

**Checkpoint**: Todas as user stories devem estar funcionalmente independentes agora

---

[Adicione mais fases de user story conforme necessário, seguindo o mesmo padrão]

---

## Fase N: Polimento e Preocupações Transversais

**Propósito**: Melhorias que afetam múltiplas user stories

- [ ] TXXX [P] Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all stories
- [ ] TXXX [P] Additional unit tests (if requested) in tests/unit/
- [ ] TXXX Security hardening
- [ ] TXXX Run quickstart.md validation

---

## Dependências e Ordem de Execução

### Dependências de Fase

- **Setup (Fase 1)**: Sem dependências — pode começar imediatamente
- **Fundacional (Fase 2)**: Depende da conclusão do Setup — BLOQUEIA todas as user stories
- **User Stories (Fase 3+)**: Todas dependem da conclusão da fase Fundacional
  - User stories podem então prosseguir em paralelo (se houver equipe)
  - Ou sequencialmente em ordem de prioridade (P1 → P2 → P3)
- **Polimento (Fase Final)**: Depende de todas as user stories desejadas estarem completas

### Dependências de User Story

- **User Story 1 (P1)**: Pode começar após Fundacional (Fase 2) — Sem dependências de outras stories
- **User Story 2 (P2)**: Pode começar após Fundacional (Fase 2) — Pode integrar com US1 mas deve ser testável independentemente
- **User Story 3 (P3)**: Pode começar após Fundacional (Fase 2) — Pode integrar com US1/US2 mas deve ser testável independentemente

### Dentro de Cada User Story

- Testes (se incluídos) DEVEM ser escritos e FALHAR antes da implementação
- Models antes de services
- Services antes de endpoints
- Implementação core antes de integração
- Story completa antes de passar para a próxima prioridade

### Oportunidades Paralelas

- Todas as tarefas de Setup marcadas [P] podem executar em paralelo
- Todas as tarefas Fundacionais marcadas [P] podem executar em paralelo (dentro da Fase 2)
- Uma vez a fase Fundacional completa, todas as user stories podem começar em paralelo (se a capacidade da equipe permitir)
- Todos os testes de uma user story marcados [P] podem executar em paralelo
- Models dentro de uma story marcados [P] podem executar em paralelo
- Diferentes user stories podem ser trabalhadas em paralelo por diferentes membros da equipe

---

## Exemplo Paralelo: User Story 1

```bash
# Launch all tests for User Story 1 together (if tests requested):
Task: "Contract test for [endpoint] in tests/contract/test_[name].py"
Task: "Integration test for [user journey] in tests/integration/test_[name].py"

# Launch all models for User Story 1 together:
Task: "Create [Entity1] model in src/models/[entity1].py"
Task: "Create [Entity2] model in src/models/[entity2].py"
```

---

## Estratégia de Implementação

### MVP Primeiro (Apenas User Story 1)

1. Complete Fase 1: Setup
2. Complete Fase 2: Fundacional (CRÍTICO - bloqueia todas as stories)
3. Complete Fase 3: User Story 1
4. **PARE e VALIDE**: Teste User Story 1 independentemente
5. Implante/demonstre se estiver pronto

### Entrega Incremental

1. Complete Setup + Fundacional → Fundação pronta
2. Adicione User Story 1 → Teste independentemente → Implante/Demonstre (MVP!)
3. Adicione User Story 2 → Teste independentemente → Implante/Demonstre
4. Adicione User Story 3 → Teste independentemente → Implante/Demonstre
5. Cada story adiciona valor sem quebrar stories anteriores

### Estratégia de Equipe Paralela

Com múltiplos desenvolvedores:

1. Equipe completa Setup + Fundacional juntos
2. Uma vez Fundacional concluída:
   - Desenvolvedor A: User Story 1
   - Desenvolvedor B: User Story 2
   - Desenvolvedor C: User Story 3
3. Stories completam e integram independentemente

---

## Notas

- Tarefas [P] = arquivos diferentes, sem dependências
- Label [Story] mapeia tarefa para user story específica para rastreabilidade
- Cada user story deve ser completável e testável independentemente
- Verifique que testes falham antes de implementar
- Faça commit após cada tarefa ou grupo lógico
- Pare em qualquer checkpoint para validar story independentemente
- Evite: tarefas vagas, conflitos no mesmo arquivo, dependências entre stories que quebram independência
