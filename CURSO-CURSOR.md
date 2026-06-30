# Usando este repositório com Cursor

Este projeto inclui **regras de projeto do Cursor** para que as diretrizes comportamentais inspiradas por Karpathy se apliquem automaticamente quando você trabalhar aqui.

## Neste repositório

1. Abra a pasta no Cursor.
2. As regras [`.cursor/rules/diretrizes-karpathy.mdc`](.cursor/rules/diretrizes-karpathy.mdc) e [`.cursor/rules/idioma-portugues.mdc`](.cursor/rules/idioma-portugues.mdc) estão comprometidas com `alwaysApply: true`.
3. Em **Settings → Rules**, confirme `diretrizes-karpathy` e `idioma-portugues`.
4. Skills do Spec Kit estão em [`.cursor/skills/speckit-*/`](.cursor/skills/) — invoque com `/speckit-plan`, `/speckit-implement`, etc.

## Use as mesmas diretrizes em outro projeto

**Cursor (recomendado):** Copie `.cursor/rules/diretrizes-karpathy.mdc` e `.cursor/rules/idioma-portugues.mdc` para `.cursor/rules/` do outro projeto.

**Outras ferramentas:** Copie [`DIRETRIZES.md`](DIRETRIZES.md) (ou [`CLAUDE.md`](CLAUDE.md) para compatibilidade com Claude Code).

## Opcional: Habilidades pessoais

Copie [`skills/diretrizes-karpathy/SKILL.md`](skills/diretrizes-karpathy/SKILL.md) para `~/.cursor/skills/`. O nome `SKILL.md` é obrigatório pelo Cursor.

## Claude Code vs Cursor

- **Claude Code:** `curl` de [`DIRETRIZES.md`](DIRETRIZES.md) conforme [`README.md`](README.md).
- **Cursor:** Use `.cursor/rules/` conforme acima. O Cursor não lê `.claude-plugin/` por padrão.

## Para contribuidores

Mantenha sincronizados ao alterar os quatro princípios:

- **[`DIRETRIZES.md`](DIRETRIZES.md)** (e alias [`CLAUDE.md`](CLAUDE.md))
- **[`.cursor/rules/diretrizes-karpathy.mdc`](.cursor/rules/diretrizes-karpathy.mdc)**
- **[`skills/diretrizes-karpathy/SKILL.md`](skills/diretrizes-karpathy/SKILL.md)**
- Metadados em **`.claude-plugin/`**

A regra **[`.cursor/rules/idioma-portugues.mdc`](.cursor/rules/idioma-portugues.mdc)** é independente, mas copie-a junto em outros projetos.

Consulte **[`docs/localizacao.md`](docs/localizacao.md)** para nomes de arquivos e exceções técnicas.
