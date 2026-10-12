# Gráfico de Gantt

**Ideal para:** planos de projeto e roadmaps — tarefas com datas explícitas de início e fim, agrupadas em fases. Use quando o leitor precisa ver, de relance, a sobreposição temporal, as trilhas paralelas e o sequenciamento de marcos.

## Convenções de layout

- **Coluna de rótulos à esquerda:** x=20–200 (180px). Nomes das tarefas em Amazon Ember 11px 600. Rótulos de fase como eyebrows em Amazon Ember Mono 7px acima de cada grupo.
- **Área da linha do tempo:** x=200–960 (760px). O eixo do tempo corre da esquerda→direita.
- **Altura da linha:** 40px por tarefa. Cada barra ocupa h=24px centralizada na linha (8px de padding superior).
- **Eixo do tempo:** rótulos de semana/mês em Amazon Ember Mono 8px em x=200+i×pitch, y=56 (logo acima da primeira linha de tarefa). Um separador em linha fina (hairline) em y=64.
- **Agrupamento por fase:** um retângulo de zona sutil (mesmo padrão da zona de arquitetura) atrás das linhas de cada fase, com um rótulo eyebrow na margem superior esquerda. Use preenchimento `rgba(35,47,62,0.02)` e traço `rgba(35,47,62,0.10)`.
- **Barra de tarefa focal:** 1 barra com preenchimento/traço accent (a entrega-chave ou a tarefa do caminho crítico). Todas as outras: preenchimento muted @ 0.15, traço muted.
- **Marcador de hoje / marco:** linha vertical tracejada opcional em `muted` na posição x da semana atual.

### Padrão de barra de tarefa

```svg
<!-- Non-focal task -->
<rect x="X_start" y="ROW_Y+8" width="DURATION_PX" height="24" rx="4"
      fill="rgba(84,91,100,0.15)" stroke="#545B64" stroke-width="1"/>
<text x="X_start+8" y="ROW_Y+25" fill="#232F3E" font-size="10" font-weight="600"
      font-family="'Amazon Ember', 'Helvetica Neue', Helvetica, Arial, sans-serif">Task name</text>

<!-- Focal task -->
<rect x="X_start" y="ROW_Y+8" width="DURATION_PX" height="24" rx="4"
      fill="rgba(255,153,0,0.12)" stroke="#EC7211" stroke-width="1"/>
<text x="X_start+8" y="ROW_Y+25" fill="#EC7211" font-size="10" font-weight="600"
      font-family="'Amazon Ember', 'Helvetica Neue', Helvetica, Arial, sans-serif">Key task</text>
```

Duração em pixels: `(end_week - start_week) × pitch`. Pitch = timeline_width / total_weeks.

## Antipadrões

- Mais de 12 tarefas (divida em subplanos ou condense em uma visão por fase).
- Mais de 5 trilhas paralelas por fase (sobreposição ilegível).
- Setas de dependência entre tarefas na v1 (adicione apenas quando essencial; use a primitiva de anotação para os rótulos).
- Datas de início/fim no rótulo da barra (coloque-as no eixo x ou em um comentário de tooltip).
- Peso visual igual para todas as barras (a tarefa focal precisa se destacar).

## Exemplos

- `assets/example-gantt.html` — minimal claro
- `assets/example-gantt-dark.html` — minimal escuro
- `assets/example-gantt-full.html` — editorial completo
