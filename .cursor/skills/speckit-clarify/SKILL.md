---
name: "speckit-clarify"
description: "Identificar áreas subespecificadas na spec da feature atual fazendo até 5 perguntas de esclarecimento altamente direcionadas e codificando as respostas de volta na spec."
compatibility: "Requer estrutura de projeto spec-kit com diretório .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/clarify.md"
---


## Entrada do Usuário

```text
$ARGUMENTS
```

Você **DEVE** considerar a entrada do usuário antes de prosseguir (se não estiver vazia).

## Verificações Pré-Execução

**Verificar hooks de extensão (antes do esclarecimento)**:
- Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se existir, leia-o e procure entradas sob a chave `hooks.before_clarify`
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue normalmente
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook opcional** (`optional: true`):
    ```
    ## Hooks de Extensão

    **Pré-Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```
  - **Hook obrigatório** (`optional: false`):
    ```
    ## Hooks de Extensão

    **Pré-Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}

    Aguarde o resultado do comando do hook antes de prosseguir para o Roteiro.
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
- Se nenhum hook estiver registrado ou `.specify/extensions.yml` não existir, ignore silenciosamente

## Roteiro

Objetivo: Detectar e reduzir ambiguidade ou pontos de decisão ausentes na especificação ativa da feature e registrar os esclarecimentos diretamente no arquivo de spec.

Nota: Este fluxo de esclarecimento deve executar (e ser concluído) ANTES de invocar `/speckit-plan`. Se o usuário declarar explicitamente que está pulando o esclarecimento (ex. spike exploratório), você pode prosseguir, mas deve avisar que o risco de retrabalho downstream aumenta.

Etapas de execução:

1. Execute `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` a partir da raiz do repositório **uma vez** (modo combinado `--json --paths-only` / `-Json -PathsOnly`). Analise campos mínimos do payload JSON:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Opcionalmente capture `IMPL_PLAN`, `TASKS` para fluxos encadeados futuros.)
   - Se a análise JSON falhar, interrompa e instrua o usuário a reexecutar `/speckit-specify` ou verificar o ambiente da branch da feature.
   - Para aspas simples em args como "I'm Groot", use sintaxe de escape: ex. 'I'\''m Groot' (ou aspas duplas se possível: "I'm Groot").

2. **SE EXISTIR**: Carregue `.specify/memory/constitution.md` para princípios do projeto e restrições de governança.

3. Carregue o arquivo de spec atual. Execute uma varredura estruturada de ambiguidade e cobertura usando esta taxonomia. Para cada categoria, marque status: Claro / Parcial / Ausente. Produza um mapa interno de cobertura usado para priorização (não produza o mapa bruto a menos que nenhuma pergunta será feita).

   Escopo Funcional e Comportamento:
   - Objetivos principais do usuário e critérios de sucesso
   - Declarações explícitas de fora de escopo
   - Diferenciação de papéis / personas de usuário

   Domínio e Modelo de Dados:
   - Entidades, atributos, relacionamentos
   - Regras de identidade e unicidade
   - Transições de ciclo de vida/estado
   - Premissas de volume de dados / escala

   Fluxo de Interação e UX:
   - Jornadas / sequências críticas do usuário
   - Estados de erro/vazio/carregamento
   - Notas de acessibilidade ou localização

   Atributos de Qualidade Não Funcionais:
   - Performance (latência, metas de throughput)
   - Escalabilidade (horizontal/vertical, limites)
   - Confiabilidade e disponibilidade (uptime, expectativas de recuperação)
   - Observabilidade (logging, métricas, sinais de tracing)
   - Segurança e privacidade (authN/Z, proteção de dados, premissas de ameaça)
   - Restrições de compliance / regulatórias (se houver)

   Integração e Dependências Externas:
   - Serviços/APIs externos e modos de falha
   - Formatos de importação/exportação de dados
   - Premissas de protocolo/versionamento

   Casos Extremos e Tratamento de Falhas:
   - Cenários negativos
   - Rate limiting / throttling
   - Resolução de conflitos (ex. edições concorrentes)

   Restrições e Tradeoffs:
   - Restrições técnicas (linguagem, armazenamento, hospedagem)
   - Tradeoffs explícitos ou alternativas rejeitadas

   Terminologia e Consistência:
   - Termos canônicos do glossário
   - Sinônimos evitados / termos obsoletos

   Sinais de Conclusão:
   - Testabilidade dos critérios de aceitação
   - Indicadores mensuráveis estilo Definition of Done

   Diversos / Placeholders:
   - Marcadores TODO / decisões não resolvidas
   - Adjetivos ambíguos ("robust", "intuitive") sem quantificação

   Para cada categoria com status Partial ou Missing, adicione uma oportunidade de pergunta candidata a menos que:
   - O esclarecimento não mudaria materialmente a implementação ou estratégia de validação
   - A informação é melhor adiada para a fase de planejamento (anote internamente)

4. Gere (internamente) uma fila priorizada de perguntas candidatas de esclarecimento (máximo 5). NÃO as produza todas de uma vez. Aplique estas restrições:
    - Máximo de 5 perguntas no total na sessão.
    - Cada pergunta deve ser respondível com OU:
       - Uma seleção de múltipla escolha curta (2–5 opções distintas e mutuamente exclusivas), OU
       - Uma resposta de uma palavra / frase curta (restrinja explicitamente: "Responda em <=5 palavras").
    - Inclua apenas perguntas cujas respostas impactam materialmente arquitetura, modelagem de dados, decomposição de tarefas, design de testes, comportamento de UX, prontidão operacional ou validação de compliance.
    - Garanta equilíbrio de cobertura de categorias: tente cobrir primeiro as categorias não resolvidas de maior impacto; evite duas perguntas de baixo impacto quando uma única área de alto impacto (ex. postura de segurança) estiver não resolvida.
    - Exclua perguntas já respondidas, preferências estilísticas triviais ou detalhes de execução em nível de plano (a menos que bloqueiem correção).
    - Prefira esclarecimentos que reduzam risco de retrabalho downstream ou previnam testes de aceitação desalinhados.
    - Se mais de 5 categorias permanecerem não resolvidas, selecione as 5 principais pela heurística (Impacto * Incerteza).

5. Loop de perguntas sequenciais (interativo):
    - Apresente EXATAMENTE UMA pergunta por vez.
    - Para perguntas de múltipla escolha:
       - **Analise todas as opções** e determine a **opção mais adequada** com base em:
          - Melhores práticas para o tipo de projeto
          - Padrões comuns em implementações similares
          - Redução de risco (segurança, performance, manutenibilidade)
          - Alinhamento com quaisquer objetivos ou restrições explícitos do projeto visíveis na spec
       - Apresente sua **opção recomendada em destaque** no topo com raciocínio claro (1-2 frases explicando por que esta é a melhor escolha).
       - Formate como: `**Recomendado:** Opção [X] - <raciocínio>`
       - Em seguida, renderize todas as opções como tabela Markdown:

       | Opção | Descrição |
       |--------|-------------|
       | A | <Descrição da Opção A> |
       | B | <Descrição da Opção B> |
       | C | <Descrição da Opção C> (adicione D/E conforme necessário, até 5) |
       | Curta | Forneça uma resposta curta diferente (<=5 palavras) (Inclua apenas se alternativa de texto livre for apropriada) |

       - Após a tabela, adicione: `Você pode responder com a letra da opção (ex.: "A"), aceitar a recomendação dizendo "sim" ou "recomendado", ou fornecer sua própria resposta curta.`
    - Para estilo de resposta curta (sem opções discretas significativas):
       - Forneça sua **resposta sugerida** com base em melhores práticas e contexto.
       - Formate como: `**Sugerido:** <sua resposta proposta> - <raciocínio breve>`
       - Em seguida, produza: `Formato: Resposta curta (<=5 palavras). Você pode aceitar a sugestão dizendo "sim" ou "sugerido", ou fornecer sua própria resposta.`
    - Após o usuário responder:
       - Se o usuário responder com "sim", "recomendado" ou "sugerido", use sua recomendação/sugestão previamente declarada como resposta.
       - Caso contrário, valide que a resposta mapeia para uma opção ou se encaixa na restrição <=5 palavras.
       - Se ambígua, peça desambiguação rápida (a contagem ainda pertence à mesma pergunta; não avance).
       - Uma vez satisfatória, registre na memória de trabalho (ainda não escreva em disco) e passe para a próxima pergunta na fila.
    - Pare de fazer perguntas quando:
       - Todas as ambiguidades críticas forem resolvidas cedo (itens restantes na fila tornam-se desnecessários), OU
       - O usuário sinalizar conclusão ("pronto", "ok", "sem mais"), OU
       - Você atingir 5 perguntas feitas.
    - Nunca revele perguntas futuras na fila com antecedência.
    - Se não existirem perguntas válidas no início, reporte imediatamente que não há ambiguidades críticas.

6. Integração após CADA resposta aceita (abordagem de atualização incremental):
    - Mantenha representação em memória da spec (carregada uma vez no início) mais o conteúdo bruto do arquivo.
    - Para a primeira resposta integrada nesta sessão:
       - Garanta que uma seção `## Esclarecimentos` exista (crie-a logo após a seção contextual/visão geral de mais alto nível conforme o template de spec se ausente).
       - Sob ela, crie (se não presente) um subheading `### Sessão YYYY-MM-DD` para hoje.
    - Acrescente uma linha com bullet imediatamente após aceitação: `- Q: <question> → A: <final answer>`.
    - Em seguida, aplique imediatamente o esclarecimento na(s) seção(ões) mais apropriada(s):
       - Ambiguidade funcional → Atualize ou adicione um bullet em Requisitos Funcionais.
       - Interação do usuário / distinção de ator → Atualize User Stories ou subseção Atores (se presente) com papel, restrição ou cenário esclarecido.
       - Forma de dados / entidades → Atualize Modelo de Dados (adicione campos, tipos, relacionamentos) preservando ordenação; anote restrições adicionadas sucintamente.
       - Restrição não funcional → Adicione/modifique critérios mensuráveis em Critérios de Sucesso > Resultados Mensuráveis (converta adjetivo vago em métrica ou meta explícita).
       - Caso extremo / fluxo negativo → Adicione um novo bullet sob Casos Extremos / Tratamento de Erros (ou crie tal subseção se o template fornecer placeholder para ela).
       - Conflito de terminologia → Normalize o termo na spec; retenha o original apenas se necessário adicionando `(anteriormente referido como "X")` uma vez.
    - Se o esclarecimento invalidar uma declaração ambígua anterior, substitua essa declaração em vez de duplicar; não deixe texto contraditório obsoleto.
    - Salve o arquivo de spec APÓS cada integração para minimizar risco de perda de contexto (sobrescrita atômica).
    - Preserve formatação: não reordene seções não relacionadas; mantenha hierarquia de headings intacta.
    - Mantenha cada esclarecimento inserido mínimo e testável (evite deriva narrativa).

7. Validação (executada após CADA escrita mais passagem final):
   - Sessão de esclarecimentos contém exatamente um bullet por resposta aceita (sem duplicatas).
   - Total de perguntas feitas (aceitas) ≤ 5.
   - Seções atualizadas não contêm placeholders vagos remanescentes que a nova resposta deveria resolver.
   - Nenhuma declaração contraditória anterior permanece (escaneie por escolhas alternativas agora inválidas removidas).
   - Estrutura markdown válida; apenas headings novos permitidos: `## Esclarecimentos`, `### Sessão YYYY-MM-DD`.
   - Consistência de terminologia: mesmo termo canônico usado em todas as seções atualizadas.

8. Escreva a spec atualizada de volta em `FEATURE_SPEC`.

9. **Revalidar Checklist de Qualidade da Spec** (se existir):
   - Verifique se `FEATURE_DIR/checklists/requirements.md` existe.
   - Se NÃO existir, pule esta etapa silenciosamente.
   - Se existir:
     1. Leia o arquivo de checklist.
     2. Identifique todas as linhas de checkbox da task list do GitHub — linhas correspondendo a `- [ ]`, `- [x]` ou `- [X]` (case-insensitive, tolerante a espaços iniciais para itens aninhados) fora de code fences. Ignore todo outro conteúdo (headings, notas, bullets sem checkbox, metadados).
     3. Para cada linha de checkbox, registre seu estado atual de marcador (marcado ou desmarcado) e texto do item em uma lista de instantâneo-anterior.
     4. Reavalie cada item de checkbox contra a spec **atualizada** (a versão recém-salva na etapa 7).
     5. Para cada item de checkbox, atualize apenas se o estado marcado/desmarcado realmente mudar:
        - Se o item agora passa e estava desmarcado: mude `[ ]` para `[x]`.
        - Se o item agora falha e estava marcado: mude `[x]`/`[X]` para `[ ]`.
        - Se o estado não mudou: deixe o marcador como está (preserve case existente para evitar diffs cosméticos).
     6. Salve o arquivo de checklist atualizado. **Alterne apenas a porção de marcador `[ ]`/`[x]` das linhas de checkbox cujo estado mudou.** Todo outro conteúdo do arquivo — headings, metadados, notas, ordenação de linhas, espaços em branco — deve permanecer inalterado para evitar diffs ruidosos.
     7. Compare o instantâneo-anterior com o estado atual para computar três listas para o Relatório de Conclusão:
        - **Recém aprovados**: itens que mudaram de desmarcado para marcado.
        - **Regressões**: itens que mudaram de marcado para desmarcado.
        - **Ainda desmarcados**: itens que permanecem desmarcados.
     8. Registre as contagens antes/depois de aprovação como itens marcados/total de itens de checkbox (ex. "12/16 → 15/16 itens aprovados").

Regras de comportamento:

- Se nenhuma ambiguidade significativa for encontrada (ou todas as perguntas potenciais seriam de baixo impacto), responda: "Nenhuma ambiguidade crítica detectada que justifique esclarecimento formal." e sugira prosseguir.
- Se o arquivo de spec estiver ausente, instrua o usuário a executar `/speckit-specify` primeiro (não crie uma nova spec aqui).
- Nunca exceda 5 perguntas feitas no total (retentativas de esclarecimento para uma única pergunta não contam como novas perguntas).
- Evite perguntas especulativas de stack técnica a menos que a ausência bloqueie clareza funcional.
- Respeite sinais de término antecipado do usuário ("pare", "pronto", "prossiga").
- Se nenhuma pergunta for feita devido a cobertura completa, produza um resumo compacto de cobertura (todas as categorias Claro) e sugira avançar.
- Se a cota for atingida com categorias de alto impacto não resolvidas remanescentes, sinalize-as explicitamente sob Adiado com justificativa.

Contexto para priorização: $ARGUMENTS

## Hooks Obrigatórios Pós-Execução

**Você DEVE completar esta seção antes de reportar a conclusão ao usuário.**

Verifique se `.specify/extensions.yml` existe na raiz do projeto.
- Se não existir, ou nenhum hook estiver registrado sob `hooks.after_clarify`, pule para o Relatório de Conclusão.
- Se existir, leia-o e procure entradas sob a chave `hooks.after_clarify`.
- Se o YAML não puder ser analisado ou for inválido, ignore a verificação de hooks silenciosamente e continue para o Relatório de Conclusão.
- Filtre hooks onde `enabled` está explicitamente como `false`. Trate hooks sem campo `enabled` como habilitados por padrão.
- Para cada hook restante, **não** tente interpretar ou avaliar expressões `condition` do hook:
  - Se o hook não tiver campo `condition`, ou for null/vazio, trate o hook como executável
  - Se o hook definir uma `condition` não vazia, pule o hook e deixe a avaliação da condição para a implementação do HookExecutor
- Ao construir comandos slash a partir de nomes de comando de hook, substitua pontos (`.`) por hífens (`-`). Por exemplo, `speckit.git.commit` → `/speckit-git-commit`.
- Para cada hook executável, produza o seguinte com base na flag `optional`:
  - **Hook obrigatório** (`optional: false`) — **Você DEVE emitir `EXECUTE_COMMAND:` para cada hook obrigatório**:
    ```
    ## Hooks de Extensão

    **Hook Automático**: {extension}
    Executando: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    Após emitir o bloco acima, você DEVE realmente invocar o hook e aguardar sua conclusão antes de continuar. Execute-o da mesma forma que executaria o comando neste agente/sessão (a invocação pode diferir do id literal `{command}` mostrado acima, por exemplo, um agente em modo skills o executa como `/skill:speckit-...` ou `$speckit-...`). Emitir o bloco sozinho não executa o hook.
  - **Hook opcional** (`optional: true`):
    ```
    ## Hooks de Extensão

    **Hook Opcional**: {extension}
    Comando: `/{command}`
    Descrição: {description}

    Instrução: {prompt}
    Para executar: `/{command}`
    ```

## Relatório de Conclusão

Reporte a conclusão (após o loop de perguntas terminar ou término antecipado):
- Número de perguntas feitas e respondidas.
- Caminho da spec atualizada.
- Seções tocadas (liste nomes).
- Status da checklist de qualidade da spec (se `FEATURE_DIR/checklists/requirements.md` foi revalidada): mostre contagens antes/depois de aprovação (ex. "Checklist de Qualidade da Spec: 12/16 → 15/16 itens aprovados") e liste quaisquer itens que mudaram de estado — tanto recém marcados (desmarcado → marcado) quanto regressões (marcado → desmarcado). Se algum item permanecer desmarcado, liste-os como áreas que precisam de atenção.
- Tabela de resumo de cobertura listando cada categoria da taxonomia com Status: Resolvido (era Parcial/Ausente e foi endereçada), Adiado (excede cota de perguntas ou melhor adequada para planejamento), Claro (já suficiente), Pendente (ainda Parcial/Ausente mas baixo impacto).
- Se Pendente ou Adiado permanecerem, recomende se prosseguir para `/speckit-plan` ou executar `/speckit-clarify` novamente mais tarde pós-plan.
- Próximo comando sugerido.

## Concluído Quando

- [ ] Ambiguidades da spec identificadas e esclarecimentos integrados no arquivo de spec
- [ ] Checklist de qualidade da spec revalidada contra spec atualizada (se `FEATURE_DIR/checklists/requirements.md` existir)
- [ ] Hooks de extensão despachados ou ignorados conforme as regras em Hooks Obrigatórios Pós-Execução acima
- [ ] Conclusão reportada ao usuário com perguntas respondidas, seções tocadas, status da checklist e resumo de cobertura
