# Contenção aninhada (Nested Containment)

**Ideal para:** hierarquia por contenção — fronteiras de escopo, cascata do CLAUDE.md, zonas de confiança, aninhamento de pastas, raio de impacto (blast radius). Externo = mais amplo, interno = mais específico.

## Convenções de layout
- 3–5 retângulos arredondados (`rx=8`), aninhados com padding interno consistente (24–32px na horizontal, 32–36px na vertical, recomendados).
- Cada nível é rotulado no canto superior esquerdo no estilo eyebrow em Amazon Ember Mono (7–8px, letter-spacing 0.14em). Os rótulos ficam sobre um retângulo de máscara da cor do paper sobre a borda superior do anel.
- Hierarquia de traços: anéis externos tênues (`rgba(..,0.30–0.45)`), progredindo para muted, depois ink, até coral no focal mais interno.
- Os preenchimentos aumentam de opacidade de fora para dentro: `rgba(..,0.015)` → `rgba(..,0.025)` → accent-tint no mais interno.
- Um glifo opcional de ícone de arquivo (retângulo com canto dobrado) dentro de cada nível sugere o conteúdo do escopo.
- Callouts em Amazon Ember itálico (veja `references/primitive-annotation.md`) — no máximo 1–2.

## Antipadrões
- Mais de 6 níveis (a informação desaparece para dentro).
- Padding irregular entre níveis — aninhamento desalinhado parece acidental.
- Conteúdo dentro dos anéis que não faz parte da hierarquia — use um diagrama irmão.
- Coral em múltiplos níveis — a hierarquia desmorona.

## Exemplos
- `assets/example-nested.html` — minimal claro
- `assets/example-nested-dark.html` — minimal escuro
- `assets/example-nested-full.html` — editorial completo
