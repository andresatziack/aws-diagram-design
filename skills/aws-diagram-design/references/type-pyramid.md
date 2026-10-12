# Pirâmide / Funil (Pyramid / Funnel)

**Ideal para:** hierarquia de necessidades, ranking de prioridades, pirâmides de valor, funis de conversão, pilhas de importância de conteúdo.

## Duas orientações — escolha uma
- **Pirâmide** (ponta para cima) — ápice estreito = mais importante / mais raro / mais valioso. A base é a mais ampla / fundamental.
- **Funil** (ponta para baixo) — extremidade estreita = conversão (menor grupo). O topo é o mais largo / público.

Não misture orientações em um mesmo diagrama.

## Convenções de layout
- 4–6 camadas. Cada camada é um trapézio construído a partir de um `<polygon>` SVG com 4 pontos.
- Altura de camada consistente (56–72px).
- As larguras diminuem linearmente da base ao ápice (pirâmide) ou do topo à base (funil). Ao mostrar dados reais de funil, as larguras devem ser honestas (proporcionais à contagem/porcentagem).
- Cada camada tem:
  - **Rótulo de nome** centralizado dentro do trapézio — Amazon Ember 12–14px 600.
  - **Sublabel** abaixo ou ao lado do nome — Amazon Ember Mono 9–10px.
  - **Anotação lateral** (direita ou esquerda) — opcional. Para funis: a porcentagem de queda vai aqui (`−40%`).
- Preenchimento: tons graduados sutis OU tudo em paper-2 com divisores em linha fina (hairline) (mais limpo). Escolha um.
- Traço: linha fina (hairline) de 1px entre camadas; silhueta externa de 1px em muted ou ink.
- **Coral em UMA camada apenas**: ápice da pirâmide, camada de conversão do funil ou gargalo crítico.
- Seta de eixo opcional na margem esquerda + rótulo em Amazon Ember Mono (`rarer ↑`, `drop-off ↓`).

## Antipadrões
- 7+ camadas (ilegível — comprima ou divida).
- Pirâmide para dados não hierárquicos (use uma árvore ou um gráfico de barras).
- Larguras desonestas (espaçamento igual falso quando as quedas são desiguais).
- Coral na camada da base (dilui o sinal de "ápice = raro").

## Exemplos
- `assets/example-pyramid.html` — minimal claro
- `assets/example-pyramid-dark.html` — minimal escuro
- `assets/example-pyramid-full.html` — editorial completo
