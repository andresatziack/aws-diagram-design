# Swimlane (raias)

**Ideal para:** processos multifuncionais, fluxos no estilo RACI, passagens de bastão (handoffs) entre fornecedores, fluxos de entrega com várias equipes.

## Convenções de layout
- Raias horizontais (ou colunas verticais) — uma por ator/equipe. Rotule cada raia na margem esquerda (ou no topo) com um eyebrow em Amazon Ember Mono.
- Divisores de raia: linhas finas (hairlines) de 1px.
- As etapas do processo são retângulos posicionados dentro da raia do ator que as executa; setas mostram o fluxo.
- Handoffs (setas que cruzam os limites das raias) são as arestas mais importantes — considere coral no handoff que introduz mais acoplamento ou latência.
- Não force o mesmo número de etapas por raia; uma raia com uma única etapa é aceitável.

## Antipadrões
- Raias sem rótulos.
- Uma etapa desenhada atravessando duas raias (escolha um único responsável).
- Setas que serpenteiam para frente e para trás — reordene as etapas para que o fluxo seja majoritariamente reto.

## Exemplos
- `assets/example-swimlane.html` — minimal claro
- `assets/example-swimlane-dark.html` — minimal escuro
- `assets/example-swimlane-full.html` — editorial completo
