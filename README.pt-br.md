# Diretrizes de Código Claude Inspiradas por Karpathy

> Confira meu novo projeto [Multica](https://github.com/multica-ai/multica) — uma plataforma de código aberto para executar e gerenciar agentes de codificação com habilidades reutilizáveis.
>
> Siga-me no X: [https://x.com/jiayuan_jy](https://x.com/jiayuan_jy)

Um único arquivo `CLAUDE.md` para melhorar o comportamento do Claude Code, derivado das [observações de Andrej Karpathy](https://x.com/karpathy/status/2015883857489522876) sobre armadilhas de codificação de LLMs.

Português Brasileiro | [English](./README.en.md) | [简体中文](./README.zh.md)

## Os Problemas

Da publicação de Andrej:

> "Os modelos fazem suposições erradas em seu nome e simplesmente prosseguem com elas sem verificar. Eles não gerenciam sua confusão, não buscam esclarecimentos, não expõem inconsistências, não apresentam compensações, não contestam quando deveriam."

> "Eles realmente gostam de complicar demais o código e as APIs, inflar abstrações, não limpam código morto... implementam uma construção inchada com mais de 1000 linhas quando 100 seriam suficientes."

> "Eles ainda às vezes alteram/removem comentários e código que não entendem suficientemente como efeitos colaterais, mesmo que ortogonais à tarefa."

## A Solução

Quatro princípios em um arquivo que abordam diretamente esses problemas:

| Princípio | Aborda |
|-----------|-----------|
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

Consulte **[EXAMPLES.pt-br.md](EXAMPLES.pt-br.md)** para exemplos detalhados de código demonstrando os quatro princípios. Versão em inglês: [EXAMPLES.md](EXAMPLES.md).

## Instalação

**Opção A: Plugin Claude Code**

O marketplace upstream (`forrestchang/andrej-karpathy-skills`) distribui a versão em inglês. Para português brasileiro, use a **Opção B** ou copie manualmente os arquivos deste repositório.

Instalação upstream (inglês):
```
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills
```

**Opção B: CLAUDE.md (por projeto, pt-BR)**

Novo projeto:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/decsters01/andrej-karpathy-skills_pt_BR/main/CLAUDE.md
```

Projeto existente (anexar):
```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/decsters01/andrej-karpathy-skills_pt_BR/main/CLAUDE.md >> CLAUDE.md
```

**Opção C: Cursor (recomendado para pt-BR)**

Clone este repositório ou copie [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) e [`.cursor/rules/idioma-pt-br.mdc`](.cursor/rules/idioma-pt-br.mdc) para o projeto. Consulte [CURSOR.md](CURSOR.md).

## Usando com Cursor

Este repositório inclui uma regra de projeto do Cursor comprometida ([`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)) para que as mesmas diretrizes se apliquem quando você abrir o projeto no Cursor. Consulte **[CURSOR.md](CURSOR.md)** para configuração, uso da regra em outros projetos e como isso se relaciona com o Claude Code.

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

Estas diretrizes são projetadas para serem mescladas com instruções específicas do projeto. Adicione-as ao seu `CLAUDE.md` existente ou crie um novo.

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
