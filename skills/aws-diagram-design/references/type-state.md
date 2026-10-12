# Máquina de estados (State Machine)

**Ideal para:** lógica de estados finitos — status de pedido, estado de autenticação, ciclo de vida de conexão, assistente de formulário (wizard), status de fila de jobs.

## Convenções de layout
- Os estados são retângulos arredondados (`rx=8`), rotulados em Amazon Ember.
- **Início**: ponto preenchido em ink (`r=6`). **Fim**: ponto com anel (contorno externo `r=8`, interno preenchido `r=5`).
- Transições: setas curvas rotuladas em Amazon Ember Mono como `event [guard] / action` (omita as seções de que não precisar).
- Auto-loops (self-loops) fazem uma curva acima do estado.
- Oriente pela direção dominante do fluxo (esquerda→direita ou cima→baixo); reorganize antes de cruzar transições.
- Coral no estado que o leitor deve notar — tipicamente o estado de erro, ou a "conclusão feliz".

## Antipadrões
- Mais transições do que estados × 2 → provavelmente são duas máquinas de estados.
- Transições "a partir de qualquer estado" desenhadas a partir de cada estado — use uma única anotação (`* → Error on timeout`).
- Transições sem rótulo (o objetivo é justamente *o que dispara isto*).

## Exemplos
- `assets/example-state.html` — minimal claro
- `assets/example-state-dark.html` — minimal escuro
- `assets/example-state-full.html` — editorial completo
