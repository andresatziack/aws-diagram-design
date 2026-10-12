# Pilha de camadas (Layer Stack)

**Ideal para:** modelo OSI, cascata do CSS, hierarquia de contexto, stack tecnológico, camadas de abstração, hierarquia de memória.

## Convenções de layout
- Faixas horizontais empilhadas verticalmente. Cada camada é um retângulo de largura total (mesmo x, mesma largura). 4–6 camadas no total.
- Altura da camada 56–72px, largura tipicamente 800–880px dentro de um viewBox de 1000px.
- Cada linha contém (esquerda→direita):
  1. **Tag de índice** na extrema esquerda (`L3`, `07`, `APPLICATION`) — eyebrow em Amazon Ember Mono 8–9px.
  2. **Nome da camada** um pouco à direita do centro-esquerda — Amazon Ember 14–16px 600.
  3. **Sublabel / nota** na extrema direita — Amazon Ember Mono 9–10px muted.
- Borda entre camadas: linha fina (hairline) de 1px `rgba(35,47,62,0.12)`. Silhueta externa de 1px em ink ou muted.
- Preenchimentos: tons sutis alternados (paper / paper-2) OU tudo em paper com divisores em linha fina (hairline). Escolha um e mantenha-o.
- Indicador de direção na margem ESQUERDA (fora da pilha): pequena seta para cima/baixo + rótulo em Amazon Ember Mono (`abstraction ↑`, `packets ↓`).
- Coral em **uma** camada focal (traço + preenchimento com tom sutil) — o gargalo, a camada que "paga o aluguel", aquela em discussão.

## Antipadrões
- Camadas que não são de fato hierárquicas (use swimlane ou arquitetura).
- Numeração pulada (L4 ausente entre L3 e L5 sem explicação).
- Cada camada de uma cor diferente — a hierarquia fica invisível.
- Alturas de camada inconsistentes sem motivo.

## Exemplos
- `assets/example-layers.html` — minimal claro
- `assets/example-layers-dark.html` — minimal escuro
- `assets/example-layers-full.html` — editorial completo
