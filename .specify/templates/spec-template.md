# Especificação da Feature: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`

**Criado**: [DATE]

**Status**: Draft

**Entrada**: Descrição do usuário: "$ARGUMENTS"

## Cenários de Usuário e Testes *(obrigatório)*

<!--
  IMPORTANTE: User stories devem ser PRIORIZADAS como jornadas de usuário ordenadas por importância.
  Cada user story/jornada deve ser TESTÁVEL INDEPENDENTEMENTE — ou seja, se você implementar apenas UMA delas,
  ainda deve ter um MVP (Minimum Viable Product) viável que entrega valor.

  Atribua prioridades (P1, P2, P3, etc.) a cada story, onde P1 é a mais crítica.
  Pense em cada story como uma fatia autônoma de funcionalidade que pode ser:
  - Desenvolvida independentemente
  - Testada independentemente
  - Implantada independentemente
  - Demonstrada aos usuários independentemente
-->

### User Story 1 - [Brief Title] (Priority: P1)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente — ex. "Pode ser totalmente testado por [ação específica] e entrega [valor específico]"]

**Cenários de Aceitação**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 - [Brief Title] (Priority: P2)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente]

**Cenários de Aceitação**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 3 - [Brief Title] (Priority: P3)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente]

**Cenários de Aceitação**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

[Adicione mais user stories conforme necessário, cada uma com prioridade atribuída]

### Casos Extremos

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com os casos extremos corretos.
-->

- O que acontece quando [boundary condition]?
- Como o sistema lida com [error scenario]?

## Requisitos *(obrigatório)*

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com os requisitos funcionais corretos.
-->

### Requisitos Funcionais

- **FR-001**: System MUST [specific capability, e.g., "allow users to create accounts"]
- **FR-002**: System MUST [specific capability, e.g., "validate email addresses"]
- **FR-003**: Users MUST be able to [key interaction, e.g., "reset their password"]
- **FR-004**: System MUST [data requirement, e.g., "persist user preferences"]
- **FR-005**: System MUST [behavior, e.g., "log all security events"]

*Exemplo de marcação de requisitos pouco claros:*

- **FR-006**: System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]
- **FR-007**: System MUST retain user data for [NEEDS CLARIFICATION: retention period not specified]

### Entidades Principais *(incluir se a feature envolve dados)*

- **[Entity 1]**: [What it represents, key attributes without implementation]
- **[Entity 2]**: [What it represents, relationships to other entities]

## Critérios de Sucesso *(obrigatório)*

<!--
  AÇÃO NECESSÁRIA: Defina critérios de sucesso mensuráveis.
  Estes devem ser agnósticos de tecnologia e mensuráveis.
-->

### Resultados Mensuráveis

- **SC-001**: [Measurable metric, e.g., "Users can complete account creation in under 2 minutes"]
- **SC-002**: [Measurable metric, e.g., "System handles 1000 concurrent users without degradation"]
- **SC-003**: [User satisfaction metric, e.g., "90% of users successfully complete primary task on first attempt"]
- **SC-004**: [Business metric, e.g., "Reduce support tickets related to [X] by 50%"]

## Premissas

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com as premissas corretas com base em defaults razoáveis
  escolhidos quando a descrição da feature não especificou certos detalhes.
-->

- [Assumption about target users, e.g., "Users have stable internet connectivity"]
- [Assumption about scope boundaries, e.g., "Mobile support is out of scope for v1"]
- [Assumption about data/environment, e.g., "Existing authentication system will be reused"]
- [Dependency on existing system/service, e.g., "Requires access to the existing user profile API"]
