# Linha do tempo (Timeline)

**Ideal para:** histórico de releases, marcos de projeto, linhas do tempo de incidentes, roadmaps, visualizações de changelog.

## Convenções de layout
- Linha de base em linha fina (hairline) horizontal atravessando o meio (`stroke-width=1`).
- Marcas de escala nos limites de tempo (trimestres, meses, sprints) com rótulos de data abaixo, em Amazon Ember Mono.
- Eventos: pequenos círculos preenchidos (`r=4`) sobre a linha de base. Os rótulos alternam acima e abaixo para evitar colisão, conectados ao círculo por uma queda em linha fina (hairline) de 1px.
- Marcos principais: círculo coral (`r=6`) + rótulo em Amazon Ember negrito.
- A escala de tempo deve ser honesta: se os intervalos não forem iguais, espace os círculos de forma desigual. Não finja espaçamento linear por estética. Quebre o eixo de forma visível se uma região estiver densa demais.

## Antipadrões
- Eventos igualmente espaçados que não são igualmente espaçados no tempo.
- Rótulos de eixo ausentes ("qual é a unidade disto?").
- Rótulos amontoados sem deslocamento vertical — ilegível.

## Exemplos
- `assets/example-timeline.html` — minimal claro
- `assets/example-timeline-dark.html` — minimal escuro
- `assets/example-timeline-full.html` — editorial completo
