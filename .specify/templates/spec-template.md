# Especificação da Feature: [FEATURE NAME]

**Ramo da Feature**: `[###-feature-name]`

**Criado**: [DATE]

**Status**: Rascunho

**Entrada**: Descrição do usuário: "$ARGUMENTS"

## Cenários de Usuário e Testes *(obrigatório)*

<!--
  IMPORTANTE: Histórias de usuário devem ser PRIORIZADAS como jornadas de usuário ordenadas por importância.
  Cada história de usuário/jornada deve ser TESTÁVEL INDEPENDENTEMENTE — ou seja, se você implementar apenas UMA delas,
  ainda deve ter um MVP (Produto Mínimo Viável) viável que entrega valor.

  Atribua prioridades (P1, P2, P3, etc.) a cada história, onde P1 é a mais crítica.
  Pense em cada história como uma fatia autônoma de funcionalidade que pode ser:
  - Desenvolvida independentemente
  - Testada independentemente
  - Implantada independentemente
  - Demonstrada aos usuários independentemente
-->

### História de Usuário 1 - [Título Breve] (Prioridade: P1)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente — ex. "Pode ser totalmente testado por [ação específica] e entrega [valor específico]"]

**Cenários de Aceitação**:

1. **Dado** [estado inicial], **Quando** [ação], **Então** [resultado esperado]
2. **Dado** [estado inicial], **Quando** [ação], **Então** [resultado esperado]

---

### História de Usuário 2 - [Título Breve] (Prioridade: P2)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente]

**Cenários de Aceitação**:

1. **Dado** [estado inicial], **Quando** [ação], **Então** [resultado esperado]

---

### História de Usuário 3 - [Título Breve] (Prioridade: P3)

[Descreva esta jornada de usuário em linguagem simples]

**Por que esta prioridade**: [Explique o valor e por que tem este nível de prioridade]

**Teste Independente**: [Descreva como isso pode ser testado independentemente]

**Cenários de Aceitação**:

1. **Dado** [estado inicial], **Quando** [ação], **Então** [resultado esperado]

---

[Adicione mais histórias de usuário conforme necessário, cada uma com prioridade atribuída]

### Casos Extremos

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com os casos extremos corretos.
-->

- O que acontece quando [condição de limite]?
- Como o sistema lida com [cenário de erro]?

## Requisitos *(obrigatório)*

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com os requisitos funcionais corretos.
-->

### Requisitos Funcionais

- **FR-001**: O sistema DEVE [capacidade específica, ex. "permitir que usuários criem contas"]
- **FR-002**: O sistema DEVE [capacidade específica, ex. "validar endereços de e-mail"]
- **FR-003**: Usuários DEVEM poder [interação principal, ex. "redefinir sua senha"]
- **FR-004**: O sistema DEVE [requisito de dados, ex. "persistir preferências do usuário"]
- **FR-005**: O sistema DEVE [comportamento, ex. "registrar todos os eventos de segurança"]

*Exemplo de marcação de requisitos pouco claros:*

- **FR-006**: O sistema DEVE autenticar usuários via [PRECISA DE ESCLARECIMENTO: método de autenticação não especificado - e-mail/senha, SSO, OAuth?]
- **FR-007**: O sistema DEVE reter dados do usuário por [PRECISA DE ESCLARECIMENTO: período de retenção não especificado]

### Entidades Principais *(incluir se a feature envolve dados)*

- **[Entidade 1]**: [O que representa, atributos principais sem implementação]
- **[Entidade 2]**: [O que representa, relacionamentos com outras entidades]

## Critérios de Sucesso *(obrigatório)*

<!--
  AÇÃO NECESSÁRIA: Defina critérios de sucesso mensuráveis.
  Estes devem ser agnósticos de tecnologia e mensuráveis.
-->

### Resultados Mensuráveis

- **SC-001**: [Métrica mensurável, ex. "Usuários podem completar a criação de conta em menos de 2 minutos"]
- **SC-002**: [Métrica mensurável, ex. "Sistema suporta 1000 usuários simultâneos sem degradação"]
- **SC-003**: [Métrica de satisfação do usuário, ex. "90% dos usuários completam a tarefa principal com sucesso na primeira tentativa"]
- **SC-004**: [Métrica de negócio, ex. "Reduzir tickets de suporte relacionados a [X] em 50%"]

## Premissas

<!--
  AÇÃO NECESSÁRIA: O conteúdo nesta seção representa placeholders.
  Preencha com as premissas corretas com base em defaults razoáveis
  escolhidos quando a descrição da feature não especificou certos detalhes.
-->

- [Premissa sobre usuários-alvo, ex. "Usuários têm conectividade estável com a internet"]
- [Premissa sobre limites de escopo, ex. "Suporte mobile está fora do escopo para v1"]
- [Premissa sobre dados/ambiente, ex. "Sistema de autenticação existente será reutilizado"]
- [Dependência de sistema/serviço existente, ex. "Requer acesso à API de perfil de usuário existente"]
