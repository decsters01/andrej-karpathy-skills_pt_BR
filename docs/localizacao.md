# Guia de Localização (pt-BR)

Este fork é **exclusivamente em português brasileiro** — conteúdo e nomes de arquivos sempre que a ferramenta permitir.

## Mapa de arquivos (português)

| Arquivo em português | Função | Notas |
|---------------------|--------|-------|
| [`LEIA-ME.md`](LEIA-ME.md) | Documentação principal | Alias de `README.md` (convenção GitHub) |
| [`DIRETRIZES.md`](DIRETRIZES.md) | Diretrizes Karpathy | Arquivo canônico |
| [`CLAUDE.md`](CLAUDE.md) | Compatibilidade Claude Code | Link simbólico → `DIRETRIZES.md` |
| [`EXEMPLOS.md`](EXEMPLOS.md) | Exemplos de código | Narrativa pt-BR |
| [`CURSO-CURSOR.md`](CURSO-CURSOR.md) | Guia de uso no Cursor | |
| [`docs/localizacao.md`](docs/localizacao.md) | Este guia | |
| `.cursor/rules/diretrizes-karpathy.mdc` | Regra Karpathy | |
| `.cursor/rules/idioma-portugues.mdc` | Regra de idioma pt-BR | |
| `skills/diretrizes-karpathy/SKILL.md` | Skill do plugin | |
| `.specify/memory/constituicao.md` | Constituição do projeto | Alias → `constitution.md` |

## Arquivos sincronizados (diretrizes Karpathy)

Ao alterar os quatro princípios, atualize todos juntos:

- `DIRETRIZES.md` (+ `CLAUDE.md` via symlink)
- `.cursor/rules/diretrizes-karpathy.mdc`
- `skills/diretrizes-karpathy/SKILL.md`
- `.claude-plugin/plugin.json` e `marketplace.json`

## Nomes que NÃO podem ser renomeados (convenção das ferramentas)

| Nome em inglês | Motivo |
|----------------|--------|
| `README.md` | GitHub exibe apenas este nome na página do repositório |
| `SKILL.md` | Cursor e Claude Code exigem este nome em pastas de skill |
| `speckit-*/` (pastas) | Slash commands `/speckit-plan` etc. dependem destes nomes |
| `spec-template.md`, `plan-template.md`, `tasks-template.md` | Scripts do Spec Kit resolvem por ID fixo |
| `constitution.md` | Skills Spec Kit referenciam este caminho |
| `spec.md`, `plan.md`, `tasks.md` | Artefatos gerados pelo workflow SDD |

## Aliases em português (symlinks)

- `LEIA-ME.md` → `README.md`
- `CLAUDE.md` → `DIRETRIZES.md`
- `constituicao.md` → `constitution.md`

## Identificadores que não traduzir no conteúdo

- Slash commands: `/speckit-plan`
- Chaves YAML/JSON: `name`, `description`, `alwaysApply`
- Nomes de ferramentas do agente: `Shell`, `Grep`
- Código-fonte nos exemplos

## Checklist

```bash
rg -i "karpathy-guidelines|idioma-pt-br|EXAMPLES\.md|CURSOR\.md|docs/i18n" README.md CURSO-CURSOR.md docs/
rg "decsters01/andrej-karpathy-skills_pt_BR" README.md
```

## Após upgrade do Spec Kit

Re-traduzir conteúdo (não renomear) em:

- `.cursor/skills/speckit-*/SKILL.md`
- `.specify/templates/*.md`
