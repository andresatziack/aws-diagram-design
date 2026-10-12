# Fluxograma (Flowchart)

**Ideal para:** lógica de decisão, algoritmos, fluxos com ramificações voltados ao usuário ("Devo…?"), roteamento de onboarding, árvores de triagem de suporte.

## Convenções de layout
- A forma carrega o tipo, não a cor:
  - **Oval** (`rx=20`) — início / fim
  - **Retângulo** (`rx=6`) — etapa / ação
  - **Losango** — decisão (≤3 saídas)
  - **Pequeno ponto preenchido em ink** (`r=4`) — ponto de junção onde os ramos se reencontram
- O fluxo corre de cima→baixo. A partir de um losango, as saídas convencionais são: Sim para a direita, Não para baixo — mas rotule toda seta de saída de qualquer forma.
- Use coral no caminho feliz (happy path) *ou* na única decisão de maior consequência — nunca em todas as decisões.
- Se duas setas precisarem se cruzar, use um pequeno salto em arco em uma delas para que o cruzamento seja legível.

## Antipadrões
- Usar cor de preenchimento para sinalizar o tipo do nó (a forma faz isso).
- Losango de decisão com 4+ saídas — refatore em losangos aninhados.
- Ramos de decisão sem rótulo.

## Exemplos
- `assets/example-flowchart.html` — minimal claro
- `assets/example-flowchart-dark.html` — minimal escuro
- `assets/example-flowchart-full.html` — editorial completo
