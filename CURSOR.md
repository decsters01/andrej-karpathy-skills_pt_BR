# Usando este repositório com Cursor

Este projeto inclui uma **regra de projeto do Cursor** para que as diretrizes comportamentais inspiradas por Karpathy se apliquem automaticamente quando você trabalhar aqui.

## Neste repositório

1. Abra a pasta no Cursor.
2. A regra [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) está comprometida com `alwaysApply: true`, então você não precisa de etapas extras de instalação.
3. No Cursor, você pode confirmar em **Settings → Rules** (ou na interface de regras do projeto), onde `karpathy-guidelines` deve aparecer.

## Use as mesmas diretrizes em outro projeto

**Cursor (recomendado):** Copie `.cursor/rules/karpathy-guidelines.mdc` para o diretório `.cursor/rules/` daquele projeto (crie as pastas se necessário). Ajuste ou mescle com regras existentes conforme desejar.

**Outras ferramentas:** Se uma pilha suportar apenas um arquivo de instrução raiz, copie [`CLAUDE.md`](CLAUDE.md) para aquele projeto (ou mescle seu conteúdo com suas instruções existentes).

## Opcional: Habilidades Pessoais do Agente

Se você quiser o mesmo conteúdo como uma habilidade reutilizável em `~/.cursor/skills`, use [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md). Você pode copiar ou criar um link simbólico para ele em seu diretório pessoal de habilidades; use qualquer layout que você use para outras habilidades.

## Claude Code vs Cursor

- **Claude Code:** Instale via marketplace de plugins e instruções do [`README.md`](README.md); o plugin expõe a habilidade deste repositório. O uso por projeto também pode depender do `CLAUDE.md`.
- **Cursor:** Use o arquivo `.cursor/rules/` comprometido conforme descrito acima. O Cursor não lê `.claude-plugin/` ou `CLAUDE.md` por padrão.

## Para contribuidores

Quando você alterar os quatro princípios, mantenha **[`CLAUDE.md`](CLAUDE.md)** e **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)** sincronizados. Se o texto da habilidade/plugin publicado dever corresponder, atualize também **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)** assim como.
