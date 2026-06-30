# Diretrizes de Código Claude Inspiradas por Karpathy

> Confira meu novo projeto [Multica](https://github.com/multica-ai/multica) — uma plataforma de código aberto para executar e gerenciar agentes de codificação com habilidades reutilizáveis.
>
> Siga-me no X: [https://x.com/jiayuan_jy](https://x.com/jiayuan_jy)

Um único arquivo [`DIRETRIZES.md`](DIRETRIZES.md) para melhorar o comportamento do Claude Code, derivado das [observações de Andrej Karpathy](https://x.com/karpathy/status/2015883857489522876) sobre armadilhas de codificação de LLMs.

> **Este repositório é exclusivamente em português brasileiro (pt-BR).**  
> `README.md` e [`LEIA-ME.md`](LEIA-ME.md) são o mesmo arquivo (convenção do GitHub).

## Os Problemas

Da publicação de Andrej:

> "Os modelos fazem suposições erradas em seu nome e simplesmente prosseguem com elas sem verificar. Eles não gerenciam sua confusão, não buscam esclarecimentos, não expõem inconsistências, não apresentam compensações, não contestam quando deveriam."

> "Eles realmente gostam de complicar demais o código e as APIs, inflar abstrações, não limpam código morto... implementam uma construção inchada com mais de 1000 linhas quando 100 seriam suficientes."

> "Eles ainda às vezes alteram/removem comentários e código que não entendem suficientemente como efeitos colaterais, mesmo que ortogonais à tarefa."

## A Solução

Quatro princípios em um arquivo que abordam diretamente esses problemas:

| Princípio | Aborda |
|-----------|---------|
| **Pense Antes de Codificar** | Suposições erradas, confusão oculta, compensações ausentes |
| **Simplicidade Primeiro** | Complicação excessiva, abstrações inchadas |
| **Mudanças Cirúrgicas** | Edições ortogonais, tocar em código que não deveria |
| **Execução Orientada a Objetivos** | Alavancagem através de testes primeiro, critérios de sucesso verificáveis |

## Os Quatro Princípios em Detalhes

### 1. Pense Antes de Codificar

**Não assuma. Não esconda confusão. Apresente compensações.**

LLMs frequentemente escolhem uma interpretação silenciosamente e prosseguem com ela. Este princípio força o raciocínio explícito:

- **Declare suposições explicitamente** — Se estiver incerto, pergunte em vez de adivinhar
- **Apresente múltiplas interpretações** — Não escolha silenciosamente quando houver ambiguidade
- **Conteste quando necessário** — Se existir uma abordagem mais simples, diga
- **Pare quando estiver confuso** — Nomeie o que não está claro e peça esclarecimento

### 2. Simplicidade Primeiro

**Mínimo de código que resolve o problema. Nada especulativo.**

Combata a tendência à superengenharia:

- Sem funcionalidades além do que foi solicitado
- Sem abstrações para código de uso único
- Sem "flexibilidade" ou "configurabilidade" que não foi solicitada
- Sem tratamento de erro para cenários impossíveis
- Se 200 linhas poderiam ser 50, reescreva

**O teste:** Um engenheiro sênior diria que isso está muito complicado? Se sim, simplifique.

### 3. Mudanças Cirúrgicas

**Toque apenas no que é necessário. Limpe apenas sua própria bagunça.**

Ao editar código existente:

- Não "melhore" código adjacente, comentários ou formatação
- Não refatore coisas que não estão quebradas
- Combine com o estilo existente, mesmo que você faria diferente
- Se notar código morto não relacionado, mencione — não o exclua

Quando suas mudanças criam órfãos:

- Remova imports/variáveis/funções que SUAS mudanças tornaram inúteis
- Não remova código morto pré-existente a menos que seja solicitado

**O teste:** Cada linha alterada deve rastrear diretamente até a solicitação do usuário.

### 4. Execução Orientada a Objetivos

**Defina critérios de sucesso. Repita até verificar.**

Transforme tarefas imperativas em objetivos verificáveis:

| Em vez de... | Transforme para... |
|--------------|-----------------|
| "Adicionar validação" | "Escrever testes para entradas inválidas, depois fazê-los passar" |
| "Corrigir o bug" | "Escrever um teste que reproduz o bug, depois fazê-lo passar" |
| "Refatorar X" | "Garantir que os testes passem antes e depois" |

Para tarefas de múltiplos passos, declare um plano breve:

```
1. [Passo] → verificar: [checagem]
2. [Passo] → verificar: [checagem]
3. [Passo] → verificar: [checagem]
```

Critérios de sucesso fortes permitem que você repita independentemente. Critérios fracos ("faça funcionar") requerem esclarecimento constante.

## Exemplos

Consulte **[EXEMPLOS.md](EXEMPLOS.md)** para exemplos detalhados de código demonstrando os quatro princípios.

## Instalação

**Opção A: Cursor (recomendado)**

Clone este repositório ou copie [`.cursor/rules/diretrizes-karpathy.mdc`](.cursor/rules/diretrizes-karpathy.mdc) e [`.cursor/rules/idioma-portugues.mdc`](.cursor/rules/idioma-portugues.mdc) para o projeto. Consulte [CURSO-CURSOR.md](CURSO-CURSOR.md).

Skills do Spec Kit para workflow SDD estão em `.cursor/skills/speckit-*/` — use `/speckit-plan`, `/speckit-implement`, etc.

**Opção B: DIRETRIZES.md (por projeto)**

Novo projeto:
```bash
curl -o DIRETRIZES.md https://raw.githubusercontent.com/decsters01/andrej-karpathy-skills_pt_BR/main/DIRETRIZES.md
```

Projeto existente (anexar):
```bash
echo "" >> DIRETRIZES.md
curl https://raw.githubusercontent.com/decsters01/andrej-karpathy-skills_pt_BR/main/DIRETRIZES.md >> DIRETRIZES.md
```

Compatibilidade Claude Code: [`CLAUDE.md`](CLAUDE.md) aponta para o mesmo conteúdo.

**Opção C: Habilidade pessoal**

Copie [`skills/diretrizes-karpathy/SKILL.md`](skills/diretrizes-karpathy/SKILL.md) para `~/.cursor/skills/` ou instale via plugin Claude Code a partir deste repositório.

> O arquivo `SKILL.md` é nome obrigatório do Cursor/Claude Code (não pode ser renomeado).

## Usando com Cursor

Este repositório inclui regras de projeto do Cursor ([`diretrizes-karpathy.mdc`](.cursor/rules/diretrizes-karpathy.mdc), [`idioma-portugues.mdc`](.cursor/rules/idioma-portugues.mdc)). Consulte **[CURSO-CURSOR.md](CURSO-CURSOR.md)** para configuração e uso em outros projetos.

## Insight Principal

De Andrej:

> "LLMs são excepcionalmente bons em repetir até atingirem objetivos específicos... Não diga a ele o que fazer, dê critérios de sucesso e observe-o ir."

O princípio "Execução Orientada a Objetivos" captura isso: transforme instruções imperativas em objetivos declarativos com loops de verificação.

## Como Saber Se Está Funcionando

Essas diretrizes estão funcionando se você vir:

- **Menos mudanças desnecessárias nos diffs** — Apenas mudanças solicitadas aparecem
- **Menos reescritas devido à complicação excessiva** — O código é simples na primeira vez
- **Perguntas de esclarecimento vêm antes da implementação** — Não depois dos erros
- **PRs limpos e mínimos** — Sem refatoração ou "melhorias" incidentais

## Personalização

Estas diretrizes são projetadas para serem mescladas com instruções específicas do projeto. Adicione-as ao seu `DIRETRIZES.md` existente ou crie um novo.

Para regras específicas do projeto, adicione seções como:

```markdown
## Diretrizes Específicas do Projeto

- Use o modo estrito do TypeScript
- Todos os endpoints de API devem ter testes
- Siga os padrões existentes de tratamento de erro em `src/utils/errors.ts`
```

## Nota de Compensação

Estas diretrizes tendem para **cautela em vez de velocidade**. Para tarefas triviais (correções simples de digitação, one-liners óbvios), use julgamento — nem toda mudança precisa de todo o rigor.

O objetivo é reduzir erros custosos em trabalho não trivial, não atrasar tarefas simples.

## Licença

MIT
