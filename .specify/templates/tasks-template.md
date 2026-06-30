---

description: "Template de lista de tarefas para implementação de feature"
---

# Tarefas: [FEATURE NAME]

**Entrada**: Documentos de design de `/specs/[###-feature-name]/`

**Pré-requisitos**: plan.md (obrigatório), spec.md (obrigatório para histórias de usuário), research.md, data-model.md, contracts/

**Testes**: Os exemplos abaixo incluem tarefas de teste. Testes são OPCIONAIS — inclua-os apenas se explicitamente solicitado na especificação da feature.

**Organização**: Tarefas são agrupadas por história de usuário para permitir implementação e teste independentes de cada história.

## Formato: `[ID] [P?] [Story] Descrição`

- **[P]**: Pode executar em paralelo (arquivos diferentes, sem dependências)
- **[Story]**: A qual história de usuário esta tarefa pertence (ex. US1, US2, US3)
- Inclua caminhos exatos de arquivo nas descrições

## Convenções de Caminho

- **Projeto único**: `src/`, `tests/` na raiz do repositório
- **App web**: `backend/src/`, `frontend/src/`
- **Mobile**: `api/src/`, `ios/src/` ou `android/src/`
- Caminhos mostrados abaixo assumem projeto único — ajuste com base na estrutura de plan.md

<!--
  ============================================================================
  IMPORTANTE: As tarefas abaixo são TAREFAS DE EXEMPLO apenas para fins de ilustração.

  O comando /speckit-tasks DEVE substituí-las por tarefas reais com base em:
  - Histórias de usuário de spec.md (com suas prioridades P1, P2, P3...)
  - Requisitos da feature de plan.md
  - Entidades de data-model.md
  - Endpoints de contracts/

  Tarefas DEVEM ser organizadas por história de usuário para que cada história possa ser:
  - Implementada independentemente
  - Testada independentemente
  - Entregue como incremento MVP

  NÃO mantenha estas tarefas de exemplo no arquivo tasks.md gerado.
  ============================================================================
-->

## Fase 1: Configuração (Infraestrutura Compartilhada)

**Propósito**: Inicialização do projeto e estrutura básica

- [ ] T001 Criar estrutura do projeto conforme plano de implementação
- [ ] T002 Inicializar projeto [language] com dependências [framework]
- [ ] T003 [P] Configurar ferramentas de linting e formatação

---

## Fase 2: Fundacional (Pré-requisitos Bloqueantes)

**Propósito**: Infraestrutura core que DEVE estar completa antes de QUALQUER história de usuário poder ser implementada

**⚠️ CRÍTICO**: Nenhum trabalho de história de usuário pode começar até esta fase estar completa

Exemplos de tarefas fundacionais (ajuste com base no seu projeto):

- [ ] T004 Configurar schema do banco de dados e framework de migrações
- [ ] T005 [P] Implementar framework de autenticação/autorização
- [ ] T006 [P] Configurar estrutura de roteamento de API e middleware
- [ ] T007 Criar models/entidades base das quais todas as histórias dependem
- [ ] T008 Configurar infraestrutura de tratamento de erros e registro
- [ ] T009 Configurar gerenciamento de configuração de ambiente

**Checkpoint**: Fundação pronta — implementação de história de usuário pode começar em paralelo agora

---

## Fase 3: História de Usuário 1 - [Título] (Prioridade: P1) 🎯 MVP

**Objetivo**: [Descrição breve do que esta história entrega]

**Teste Independente**: [Como verificar que esta história funciona por conta própria]

### Testes para História de Usuário 1 (OPCIONAL - apenas se testes solicitados) ⚠️

> **NOTA: Escreva estes testes PRIMEIRO, garanta que FALHEM antes da implementação**

- [ ] T010 [P] [US1] Teste de contrato para [endpoint] em tests/contract/test_[name].py
- [ ] T011 [P] [US1] Teste de integração para [jornada do usuário] em tests/integration/test_[name].py

### Implementação para História de Usuário 1

- [ ] T012 [P] [US1] Criar model [Entity1] em src/models/[entity1].py
- [ ] T013 [P] [US1] Criar model [Entity2] em src/models/[entity2].py
- [ ] T014 [US1] Implementar [Service] em src/services/[service].py (depende de T012, T013)
- [ ] T015 [US1] Implementar [endpoint/feature] em src/[location]/[file].py
- [ ] T016 [US1] Adicionar validação e tratamento de erros
- [ ] T017 [US1] Adicionar registro para operações da história de usuário 1

**Checkpoint**: Neste ponto, História de Usuário 1 deve estar totalmente funcional e testável independentemente

---

## Fase 4: História de Usuário 2 - [Título] (Prioridade: P2)

**Objetivo**: [Descrição breve do que esta história entrega]

**Teste Independente**: [Como verificar que esta história funciona por conta própria]

### Testes para História de Usuário 2 (OPCIONAL - apenas se testes solicitados) ⚠️

- [ ] T018 [P] [US2] Teste de contrato para [endpoint] em tests/contract/test_[name].py
- [ ] T019 [P] [US2] Teste de integração para [jornada do usuário] em tests/integration/test_[name].py

### Implementação para História de Usuário 2

- [ ] T020 [P] [US2] Criar model [Entity] em src/models/[entity].py
- [ ] T021 [US2] Implementar [Service] em src/services/[service].py
- [ ] T022 [US2] Implementar [endpoint/feature] em src/[location]/[file].py
- [ ] T023 [US2] Integrar com componentes da História de Usuário 1 (se necessário)

**Checkpoint**: Neste ponto, Histórias de Usuário 1 E 2 devem funcionar independentemente

---

## Fase 5: História de Usuário 3 - [Título] (Prioridade: P3)

**Objetivo**: [Descrição breve do que esta história entrega]

**Teste Independente**: [Como verificar que esta história funciona por conta própria]

### Testes para História de Usuário 3 (OPCIONAL - apenas se testes solicitados) ⚠️

- [ ] T024 [P] [US3] Teste de contrato para [endpoint] em tests/contract/test_[name].py
- [ ] T025 [P] [US3] Teste de integração para [jornada do usuário] em tests/integration/test_[name].py

### Implementação para História de Usuário 3

- [ ] T026 [P] [US3] Criar model [Entity] em src/models/[entity].py
- [ ] T027 [US3] Implementar [Service] em src/services/[service].py
- [ ] T028 [US3] Implementar [endpoint/feature] em src/[location]/[file].py

**Checkpoint**: Todas as histórias de usuário devem estar funcionalmente independentes agora

---

[Adicione mais fases de história de usuário conforme necessário, seguindo o mesmo padrão]

---

## Fase N: Polimento e Preocupações Transversais

**Propósito**: Melhorias que afetam múltiplas histórias de usuário

- [ ] TXXX [P] Atualizações de documentação em docs/
- [ ] TXXX Limpeza e refatoração de código
- [ ] TXXX Otimização de performance em todas as histórias
- [ ] TXXX [P] Testes unitários adicionais (se solicitados) em tests/unit/
- [ ] TXXX Reforço de segurança
- [ ] TXXX Executar validação de quickstart.md

---

## Dependências e Ordem de Execução

### Dependências de Fase

- **Configuração (Fase 1)**: Sem dependências — pode começar imediatamente
- **Fundacional (Fase 2)**: Depende da conclusão da Configuração — BLOQUEIA todas as histórias de usuário
- **Histórias de Usuário (Fase 3+)**: Todas dependem da conclusão da fase Fundacional
  - Histórias de usuário podem então prosseguir em paralelo (se houver equipe)
  - Ou sequencialmente em ordem de prioridade (P1 → P2 → P3)
- **Polimento (Fase Final)**: Depende de todas as histórias de usuário desejadas estarem completas

### Dependências de História de Usuário

- **História de Usuário 1 (P1)**: Pode começar após Fundacional (Fase 2) — Sem dependências de outras histórias
- **História de Usuário 2 (P2)**: Pode começar após Fundacional (Fase 2) — Pode integrar com US1 mas deve ser testável independentemente
- **História de Usuário 3 (P3)**: Pode começar após Fundacional (Fase 2) — Pode integrar com US1/US2 mas deve ser testável independentemente

### Dentro de Cada História de Usuário

- Testes (se incluídos) DEVEM ser escritos e FALHAR antes da implementação
- Models antes de services
- Services antes de endpoints
- Implementação core antes de integração
- História completa antes de passar para a próxima prioridade

### Oportunidades Paralelas

- Todas as tarefas de Configuração marcadas [P] podem executar em paralelo
- Todas as tarefas Fundacionais marcadas [P] podem executar em paralelo (dentro da Fase 2)
- Uma vez a fase Fundacional completa, todas as histórias de usuário podem começar em paralelo (se a capacidade da equipe permitir)
- Todos os testes de uma história de usuário marcados [P] podem executar em paralelo
- Models dentro de uma história marcados [P] podem executar em paralelo
- Diferentes histórias de usuário podem ser trabalhadas em paralelo por diferentes membros da equipe

---

## Exemplo Paralelo: História de Usuário 1

```bash
# Iniciar todos os testes da História de Usuário 1 juntos (se testes solicitados):
Task: "Teste de contrato para [endpoint] em tests/contract/test_[name].py"
Task: "Teste de integração para [jornada do usuário] em tests/integration/test_[name].py"

# Iniciar todos os models da História de Usuário 1 juntos:
Task: "Criar model [Entity1] em src/models/[entity1].py"
Task: "Criar model [Entity2] em src/models/[entity2].py"
```

---

## Estratégia de Implementação

### MVP Primeiro (Apenas História de Usuário 1)

1. Conclua a Fase 1: Configuração
2. Conclua a Fase 2: Fundacional (CRÍTICO - bloqueia todas as histórias)
3. Conclua a Fase 3: História de Usuário 1
4. **PARE e VALIDE**: Teste História de Usuário 1 independentemente
5. Implante/demonstre se estiver pronto

### Entrega Incremental

1. Conclua Configuração + Fundacional → Fundação pronta
2. Adicione História de Usuário 1 → Teste independentemente → Implante/Demonstre (MVP!)
3. Adicione História de Usuário 2 → Teste independentemente → Implante/Demonstre
4. Adicione História de Usuário 3 → Teste independentemente → Implante/Demonstre
5. Cada história adiciona valor sem quebrar histórias anteriores

### Estratégia de Equipe Paralela

Com múltiplos desenvolvedores:

1. Equipe conclui Configuração + Fundacional juntos
2. Uma vez Fundacional concluída:
   - Desenvolvedor A: História de Usuário 1
   - Desenvolvedor B: História de Usuário 2
   - Desenvolvedor C: História de Usuário 3
3. Histórias completam e integram independentemente

---

## Notas

- Tarefas [P] = arquivos diferentes, sem dependências
- Rótulo [Story] mapeia tarefa para história de usuário específica para rastreabilidade
- Cada história de usuário deve ser completável e testável independentemente
- Verifique que testes falham antes de implementar
- Faça commit após cada tarefa ou grupo lógico
- Pare em qualquer checkpoint para validar história independentemente
- Evite: tarefas vagas, conflitos no mesmo arquivo, dependências entre histórias que quebram independência
