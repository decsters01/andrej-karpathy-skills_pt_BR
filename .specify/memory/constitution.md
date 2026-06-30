# Constituição do Fork pt-BR — Diretrizes Karpathy

## Princípios Fundamentais

### I. Português Brasileiro como Idioma Padrão

Todo conteúdo voltado ao usuário e ao agente deve estar em português brasileiro (pt-BR). Isso inclui READMEs, diretrizes, skills, templates, descrições de plugin e mensagens de workflow. Identificadores técnicos (nomes de skills, slash commands, chaves YAML/JSON, caminhos de arquivo, comandos shell) permanecem em inglês para compatibilidade com ferramentas.

### II. Preservação de Identificadores Técnicos

Nunca traduzir: IDs de skill/plugin (`karpathy-guidelines`, `speckit-plan`), invocações (`/speckit-plan`), chaves de configuração (`name`, `description`, `alwaysApply`), URLs GitHub, nomes de ferramentas do Cursor (`Shell`, `Grep`, etc.). Traduzir apenas texto narrativo, instruções e descrições.

### III. Sincronização de Artefatos

Os quatro arquivos de diretrizes Karpathy devem permanecer sincronizados em conteúdo (diferenças apenas estruturais de frontmatter):

- `CLAUDE.md`
- `.cursor/rules/karpathy-guidelines.mdc`
- `skills/karpathy-guidelines/SKILL.md`
- Metadados do plugin em `.claude-plugin/`

A regra de idioma em `.cursor/rules/idioma-pt-br.mdc` aplica-se em conjunto com as diretrizes Karpathy.

### IV. Instalação a Partir do Fork

Instruções de instalação devem apontar para `decsters01/andrej-karpathy-skills_pt_BR`. O `README.md` é a documentação principal em pt-BR. Não há versões alternativas em outros idiomas neste fork.

### V. Simplicidade e Mudanças Cirúrgicas

Seguir as diretrizes Karpathy: mínimo de código e documentação que resolve o problema; tocar apenas arquivos necessários à localização; não refatorar conteúdo não relacionado.

## Restrições de Localização

- Exemplos de código em `EXAMPLES*.md` permanecem em inglês (código-fonte); narrativa em pt-BR.
- Skills do Spec Kit (`speckit-*`): traduzir corpo e `description` no frontmatter; preservar nomes de pasta e invocações.
- Templates em `.specify/templates/`: traduzir para pt-BR mantendo placeholders técnicos intactos.

## Fluxo de Desenvolvimento

1. Usar workflow Spec Kit (`/speckit-constitution` → `/speckit-specify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`) para mudanças estruturais.
2. Verificar links i18n nos READMEs após cada alteração.
3. Validar ausência de texto inglês residual em arquivos de diretriz com busca por frases típicas.

## Governança

Esta constituição tem precedência sobre práticas ad hoc de tradução. Emendas exigem atualização deste arquivo e sincronização dos artefatos dependentes. Toda PR deve verificar conformidade com os princípios de idioma e sincronização.

**Versão**: 1.0.0 | **Ratificada**: 2026-06-30 | **Última Emenda**: 2026-06-30
