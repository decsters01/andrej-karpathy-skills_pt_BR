# Constituição do Fork pt-BR — Diretrizes Karpathy

## Princípios Fundamentais

### I. Português Brasileiro como Idioma Padrão

Todo conteúdo e nomes de arquivos devem estar em português brasileiro sempre que a ferramenta permitir. Identificadores técnicos obrigatórios (slash commands, `SKILL.md`, pastas `speckit-*`) permanecem em inglês.

### II. Preservação de Identificadores Técnicos

Nunca traduzir: pastas `speckit-*`, invocações `/speckit-plan`, chaves YAML/JSON, nome `SKILL.md`, templates `*-template.md` do Spec Kit, artefatos `spec.md`/`plan.md`/`tasks.md`.

### III. Sincronização de Artefatos

Manter sincronizados:

- `DIRETRIZES.md` (canônico; `CLAUDE.md` é alias)
- `.cursor/rules/diretrizes-karpathy.mdc`
- `skills/diretrizes-karpathy/SKILL.md`
- Metadados do plugin em `.claude-plugin/`

Regra de idioma: `.cursor/rules/idioma-portugues.mdc`

### IV. Instalação a Partir do Fork

Instruções apontam para `decsters01/andrej-karpathy-skills_pt_BR`. Documentação em `LEIA-ME.md` / `README.md`.

### V. Simplicidade e Mudanças Cirúrgicas

Seguir as diretrizes Karpathy em mudanças de localização.

## Restrições de Localização

- Código em `EXEMPLOS.md` permanece em inglês; narrativa em pt-BR.
- Skills Spec Kit: traduzir conteúdo; preservar nomes de pasta e `SKILL.md`.
- Templates `.specify/templates/`: traduzir conteúdo; preservar nomes de arquivo.

## Governança

**Versão**: 1.1.0 | **Ratificada**: 2026-06-30 | **Última Emenda**: 2026-06-30
