# Usando este repositório com Cursor

Este projeto inclui uma **regra de projeto do Cursor** para que as diretrizes comportamentais inspiradas por Karpathy se apliquem automaticamente quando você trabalhar aqui.

## Neste repositório

1. Abra a pasta no Cursor.
2. As regras [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) e [`.cursor/rules/idioma-pt-br.mdc`](.cursor/rules/idioma-pt-br.mdc) estão comprometidas com `alwaysApply: true`, então você não precisa de etapas extras de instalação.
3. No Cursor, você pode confirmar em **Settings → Rules** (ou na interface de regras do projeto), onde `karpathy-guidelines` e `idioma-pt-br` devem aparecer.
4. Skills do Spec Kit (workflow SDD) estão em [`.cursor/skills/speckit-*/`](.cursor/skills/) — invoque com `/speckit-plan`, `/speckit-implement`, etc.

## Use as mesmas diretrizes em outro projeto

**Cursor (recomendado):** Copie `.cursor/rules/karpathy-guidelines.mdc` e `.cursor/rules/idioma-pt-br.mdc` para o diretório `.cursor/rules/` daquele projeto (crie as pastas se necessário). Ajuste ou mescle com regras existentes conforme desejar.

**Outras ferramentas:** Se uma pilha suportar apenas um arquivo de instrução raiz, copie [`CLAUDE.md`](CLAUDE.md) para aquele projeto (ou mescle seu conteúdo com suas instruções existentes).

## Opcional: Habilidades Pessoais do Agente

Se você quiser o mesmo conteúdo como uma habilidade reutilizável em `~/.cursor/skills`, use [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md). Você pode copiar ou criar um link simbólico para ele em seu diretório pessoal de habilidades; use qualquer layout que você use para outras habilidades.

## Claude Code vs Cursor

- **Claude Code:** Use `curl` do fork `decsters01/andrej-karpathy-skills_pt_BR` conforme [`README.md`](README.md).
- **Cursor:** Use os arquivos `.cursor/rules/` comprometidos conforme descrito acima. O Cursor não lê `.claude-plugin/` ou `CLAUDE.md` por padrão.

## Para contribuidores

Quando você alterar os quatro princípios, mantenha sincronizados:

- **[`CLAUDE.md`](CLAUDE.md)**
- **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)**
- **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)**
- Metadados do plugin em **`.claude-plugin/`** (se o texto publicado deve corresponder)

A regra de idioma **[`.cursor/rules/idioma-pt-br.mdc`](.cursor/rules/idioma-pt-br.mdc)** é independente dos quatro princípios, mas deve ser copiada junto ao usar as diretrizes em outros projetos.

Consulte **[`docs/i18n.md`](docs/i18n.md)** para o guia completo de localização e identificadores que não devem ser traduzidos.
