# Organograma / Mapa de responsabilidades (Org Chart)

**Ideal para:** equipes humanas, equipes de agentes, mapas de escalonamento de suporte, responsabilidade por papéis, mapas de roteamento e qualquer hierarquia em que o leitor precise saber *quem é responsável por quê*, e não apenas a estrutura pai → filho.

Use **Org Chart** em vez de **Tree** quando os nós forem pessoas, agentes, equipes, papéis ou responsáveis com prestação de contas. Uma árvore mostra hierarquia genérica. Um organograma mostra responsabilidade, caminhos de acionamento e lacunas de cobertura.

## Convenções de layout
- Responsável raiz ou porta de entrada no topo, ao centro. Use um único nó focal em coral para a pessoa/equipe/agente que recebe trabalho ambíguo.
- Os nós do nível 1 são departamentos, pods, filas ou buckets primários de roteamento. Mantenha-os alinhados horizontalmente.
- Os nós do nível 2 são responsáveis ou especialistas. Se houver mais de 8 especialistas, agrupe-os sob nós de pod em vez de criar uma única fileira gigante.
- Use conectores ortogonais: queda vertical a partir do pai → barramento horizontal → quedas verticais até os filhos. Nada de linhas diagonais.
- Cada nó deve responder a três perguntas quando houver espaço:
  1. **Nome** — papel/pessoa/agente legível por humanos, em Amazon Ember.
  2. **Como acionar** — handle do Slack, fila, prefixo de issue ou gatilho, em Amazon Ember Mono.
  3. **Escopo** — 2–4 palavras concisas de responsabilidade, não um parágrafo.
- Mostre responsáveis fora do Slack / ainda não ativos com o estilo opcional tracejado em vez de ocultá-los. Rotas ausentes são operacionalmente importantes.
- Coloque regras de escalonamento / aprovação em um pequeno callout lateral ou em uma faixa de rodapé, não como nós extras do organograma.

## Tratamentos de nós
- **Porta de entrada / central de comando:** tratamento focal (`accent-tint` + `accent`).
- **Equipe / pod / departamento:** tratamento de backend (branco + `ink`).
- **Agente / responsável individual:** tratamento de store ou externo, dependendo de estar ativo no sistema.
- **Lacuna / precisa de configuração:** tratamento opcional tracejado.
- **Portão de aprovação:** tratamento de segurança, separado da hierarquia de reporte.

## Orçamento de complexidade (limite)
- Máximo de nós visíveis no organograma: 12. Se houver mais, crie um organograma de visão geral mais organogramas de detalhe separados por pod.
- Profundidade máxima: 4 níveis.
- Máximo de subordinados diretos sob um pai: 5. Se houver mais, introduza nós de agrupamento.
- Máximo de nós em coral: 1. A função do organograma é clareza, não destacar tudo.
- Máximo de callouts laterais: 2.

## Antipadrões
- Usar um swimlane quando a pergunta real do usuário é "quem faz o quê?". Swimlanes explicam processo; organogramas explicam responsabilidade.
- Desenhar cada pessoa/agente como uma caixa idêntica. Isso esconde a porta de entrada, os especialistas, as lacunas e os caminhos de escalonamento.
- Espremer descrições de cargo completas dentro dos nós. Mantenha as frases de escopo curtas e mova os detalhes para cards de resumo abaixo.
- Mostrar agentes indisponíveis / ainda não conectados como responsáveis ativos normais. Use o estilo opcional tracejado para que as lacunas fiquem visíveis.
- Repetir handles do Slack nos parágrafos do corpo quando um sublabel do nó pode carregar o caminho de acionamento.
- Legendas flutuando na área do organograma. Use a faixa de legenda padrão na parte inferior.

## Exemplos
- `assets/example-org-chart.html` — minimal claro
- `assets/example-org-chart-dark.html` — minimal escuro
- `assets/example-org-chart-full.html` — editorial completo
