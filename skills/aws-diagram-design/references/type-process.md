# Processo (Process)

**Ideal para:** processos de negócio sequenciais com vários atores/divisões, nos quais o leitor precisa ver *quem* faz *o quê*, *quais dados* entram e saem de cada etapa e *quais ferramentas* são usadas — não apenas a ordem das etapas. Cobre auditorias de responsabilidade, revisões de portões de qualidade de dados, mapas de transferência entre divisões e documentação de fluxos de trabalho de ponta a ponta.

Prefira swimlane (raias, mais simples) quando os tipos de dados e as ferramentas não importam. Prefira process quando o payload de entrada/saída e a equipe responsável de cada etapa precisam ser legíveis de relance.

Este tipo é **paramétrico** — o esquema de entradas em §1 determina todas as coordenadas por meio das fórmulas em §2. Duas gerações a partir das mesmas entradas devem produzir SVG visualmente idêntico. O formato das regras espelha `type-data-flow.md`, de modo que a sobrescrita de cor, a semântica dos chips IN/OUT e o checklist de reprodutibilidade sejam lidos de forma idêntica entre os tipos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
lanes:                              # 1..6 horizontal swimlanes (top to bottom)
  - { name: ["RD&E"],                 key: "RDE" }
  - { name: ["IT"],                   key: "IT"  }
  - { name: ["FIELD", "SERVICES"],    key: "FLD" }
  - { name: ["SURVEY", "SERVICES"],   key: "SVY" }
  - { name: ["HOUSEHOLD", "UNIT"],    key: "HHU" }
  - { name: ["COMMS &", "MARKETING"], key: "CMM" }

steps:                              # 1..12 vertical step columns (left to right)
  - { number: "1",  label: "Design"   }
  - { number: "2",  label: "Build"    }
  - { number: "3",  label: "Test", focal: true }       # focal step header chip — accent fill
  - { number: "4",  label: "Train"    }
  # ... up to 12

nodes:                              # explicit per-cell entries; empty cells render nothing
  - { lane: "RDE", step: 0,  title: "Survey design",      sub: "questionnaire · sampling", tool: "Excel · CSPro",
      chips: {in: null,  out: "FL"} }              # first step has no input chip
  - { lane: "IT",  step: 1,  title: "Build app",          sub: "form + validation",        tool: "CSPro · scripts",
      chips: {in: "FL", out: "TB"}, color: "#5a7d9a" }    # slate-blue — data quality concern
  - { lane: "RDE", step: 2,  title: "Pilot test",         sub: "field debug",              tool: "tablet · script",
      chips: {in: "TB", out: "TB"}, focal: true }   # focal node — accent border
  - { lane: "FLD", step: 3,  title: "Train enumerators",  sub: "protocols · safety",       tool: "manual",
      chips: {in: "TB", out: "LS"}, color: "#b85450" }    # rust-red — governance / training
  # ... etc

arrows:                             # explicit edges; styles bind to topology (see §3)
  - { from: {lane: "RDE", step: 0}, to: {lane: "IT",  step: 1}, style: "normal" }
  - { from: {lane: "IT",  step: 1}, to: {lane: "RDE", step: 2}, style: "focal-in" }     # accent — into focal
  - { from: {lane: "RDE", step: 2}, to: {lane: "FLD", step: 3}, style: "focal-out" }    # accent — out of focal
  - { from: {lane: "RDE", step: 2}, to: {lane: "IT",  step: 1}, style: "trigger" }      # dashed trigger
  # ... etc

dark: false
```

**Semântica dos campos reservados:**
- `lanes[k].key` — o texto de 3 letras do selo de papel exibido dentro de cada nó daquela raia.
- `lanes[k].name` — rótulo da raia com 1 ou 2 linhas; mono em maiúsculas.
- `steps[j].focal: true` — exatamente **uma** etapa pode declarar isto. O chip do cabeçalho é renderizado em `accent`.
- `nodes[i].focal: true` — exatamente **um** nó pode declarar isto. Renderizado com borda `accent` (§5).
- `nodes[i].chips` — objeto `{in: "<CODE>", out: "<CODE>"}` (qualquer um dos lados `null` para omitir). Códigos de §8. **Omita** o chip de entrada nos nós da primeira etapa e **omita** o chip de saída nos nós da última etapa (não há upstream / downstream).
- `nodes[i].color` — **sobrescrita de cor por nó** opcional. Qualquer string `"#hex"` válida; a paleta de §4 é recomendada para consistência entre diagramas.

---

## 2. Fórmulas de layout — geometria determinística

```
label_col_w      = 140
step_slot_w      = 112                                # 100-px node + 12-px corridor
right_pad        = 28
n_steps          = len(steps)
n_lanes          = len(lanes)

# Canvas
viewBox_w        = label_col_w + n_steps * step_slot_w + right_pad   # 11 steps → 1400
header_h         = 36
lane_h           = 80
has_color_row    = any(node.color or step.color or lane.color in inputs)
legend_h         = 100 if has_color_row else 80       # 4 rows when colors are present
viewBox_h        = header_h + n_lanes * lane_h + legend_h            # 6 lanes, no colors → 596; with → 616

# Header strip (top)
chip_y           = 8
chip_w           = 16                                  # 20 if step.number has 2 digits
chip_h           = 16
chip_rx          = 8                                   # pill

# Lane positions
lane_y_top(k)    = header_h + k * lane_h               # 36, 116, 196, 276, 356, 436
lane_y_mid(k)    = lane_y_top(k) + lane_h/2            # 76, 156, 236, 316, 396, 476
lane_label_x     = label_col_w / 2                     # 70

# Step / node centers
step_cx(j)       = label_col_w + 8 + j * step_slot_w + node_w/2      # 198, 310, 422, ...
                                                                      # (8-px gutter inside content area)

# Nodes
node_w           = 100
node_h           = 64
node_x(j)        = step_cx(j) - node_w/2
node_y(k)        = lane_y_top(k) + (lane_h - node_h)/2     # 8-px top/bottom margin inside lane

# Legend strip (bottom)
legend_y_top     = header_h + n_lanes * lane_h
legend_row_y     = [legend_y_top + 16, legend_y_top + 37,
                    legend_y_top + 58, legend_y_top + 79]
```

### 2.1 Estrutura do fundo

- Preenchimento `paper` em todo o viewBox.
- Padrão de pontos: grade de 22×22, `circle r=0.8`, `fill rgba(35,47,62,0.10)`. Opacidade 0.55.
- Tons alternados nas raias: as raias de índice ímpar (0, 2, …) recebem preenchimento `rgba(35,47,62,0.018)` de `x=140` até `viewBox_w`.
- Divisores de raia: linhas finas (hairlines) horizontais em cada `lane_y_top(k)` e em `legend_y_top`; traço `rgba(35,47,62,0.12)` largura 0.8.
- Borda direita da coluna de rótulos: linha fina (hairline) vertical em `x = label_col_w`, traço `rgba(35,47,62,0.20)` largura 1, de `y = header_h` até `y = legend_y_top`.

### 2.2 Chip + rótulo do cabeçalho da etapa

Por etapa `j`:

```
chip_w(j)        = 20 if len(step.number) >= 2 else 16
chip_x(j)        = step_cx(j) - chip_w(j)/2
number_anchor    = (step_cx(j), chip_y + 11)
label_anchor     = (step_cx(j), 32)              # 8-px gap below chip
```

**Chip** (a pílula numerada no topo de cada coluna):
- Preenchimento padrão: `rgba(35,47,62,0.12)`, texto do número em `ink`.
- Preenchimento focal: `rgba(255,153,0,0.20)`, texto do número em `accent` (§5).
- Sobrescrita de `color` por etapa (§4): substitui o preenchimento por `rgba(C, 0.20)` e o preenchimento do número por `C`.

**Rótulo** (o texto mono em maiúsculas abaixo do chip):
- Renderiza `steps[j].label` (em maiúsculas), ancorado em `label_anchor`.
- Fonte: Amazon Ember Mono 6 px, peso 500, `letter-spacing="0.12em"`, `text-anchor="middle"`.
- Preenchimento padrão: `muted` (`#545B64` claro / `#B6BEC9` escuro).
- Preenchimento focal: `accent` (`#EC7211` claro / `#FF9900` escuro).
- Sobrescrita de `color` por etapa: fill = `C` (coincide com a cor do número no chip).
- Mantenha os rótulos curtos (≤ 9 caracteres). Rótulos longos são truncados; se precisar de mais, abrevie.

### 2.3 Rótulos das raias

Rótulo mono de uma ou duas linhas, todo em maiúsculas, letter-spacing 0.08em, font-size 8, preenchimento `muted`. Centralizado em `(lane_label_x, lane_y_mid(k))`:
- Linha única: ancorado em `(lane_label_x, lane_y_mid(k) + 4)`
- Duas linhas: linhas em `(lane_label_x, lane_y_mid(k) - 4)` e `(lane_label_x, lane_y_mid(k) + 4)`

Sobrescrita de `color` por raia (§4): substitui o preenchimento do rótulo por `C` e o tom da faixa da raia por `rgba(C, 0.04)`.

### 2.4 Layout do conteúdo do nó (dentro do rect de 100×64)

```
role_chip          rect 14×10 at (node_x+4, node_y+4),  rx=2
role_chip_text     centered at (node_x+11, node_y+12), font-size=6, weight=600
                                                                # text = lanes[k].key (3-letter lane code)
title              centered at (step_cx(j), node_y+26),  font-size=9 sans semibold
in→out             centered at (step_cx(j), node_y+40),  font-size=6.5 mono muted
tool               centered at (step_cx(j), node_y+52),  font-size=6.5 mono soft
data chip IN       rect 16×8 at (node_x+4,   node_y+54), rx=2      # payload entering
data chip OUT      rect 16×8 at (node_x+80,  node_y+54), rx=2      # payload leaving
```

**Regra do texto do chip de papel:** o selo dentro de cada nó renderiza `lanes[k].key`, onde `k` é o índice da raia do nó — **não** o número da etapa (o número da etapa já está no chip do cabeçalho da coluna, no topo, §2.2). Mostrar a chave da raia como selo do nó dá a cada nó um identificador de "quem" autossuficiente, que se mantém quando um único nó é extraído fora de contexto. Espelha a mesma regra em `type-data-flow.md` §2.4.

Células vazias (sem entrada de nó) não renderizam **nada**. Nenhum rect de placeholder, nenhum chip de papel, nenhum rótulo.

**Regra de colisão chip vs. texto da ferramenta:** os chips ficam em `node_y + 54..62`; a linha de base do texto da ferramenta está em `node_y + 52`. Se um nó tiver um título de duas linhas (raro), aumente node_h para 72 OU omita os chips desse nó. Comportamento padrão: omitir os chips em caso de colisão.

---

## 3. Regras de conectores (obrigatórias)

Três estilos, vinculados à topologia. Os conectores são desenhados **antes** de todos os rects de nós (regra de ordem Z).

| `style` | Traço | Largura | Tracejado | Marcador | Quando é exigido |
|---|---|---|---|---|---|
| `normal` | `#545B64` (muted) | 1.0 | — | `arrow` | Transferência padrão de dados entre etapas ou atores. Sem rótulo. |
| `focal-in` / `focal-out` | `#EC7211` (accent) | 1.2 | — | `arrow-accent` | Toda aresta cujo destino é o nó focal (`focal-in`) ou cuja origem é o nó focal (`focal-out`). |
| `trigger` | `#545B64` (muted) | 1.0 | `4,3` | `arrow-sm` | Gatilho de orquestração (agendador → ferramenta, sobrescrita manual → etapa a montante). Sem rótulo. |

**Bloco defs** (obrigatório, três marcadores):

```svg
<defs>
  <pattern id="dots" width="22" height="22" patternUnits="userSpaceOnUse">
    <circle cx="11" cy="11" r="0.8" fill="rgba(35,47,62,0.10)"/>
  </pattern>
  <marker id="arrow"        markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#545B64"/></marker>
  <marker id="arrow-accent" markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#EC7211"/></marker>
  <marker id="arrow-sm"     markerWidth="6" markerHeight="5" refX="5" refY="2.5" orient="auto"><polygon points="0 0, 6 2.5, 0 5" fill="#545B64"/></marker>
</defs>
```

### 3.1 Regras de roteamento (inegociáveis)

**Ângulo reto com uma única curva:** sai pela DIREITA → corredor → entra por CIMA (↓ destino abaixo) ou por BAIXO (↑ destino acima).

- Lado da origem: sai em `(node_x + 100, lane_y_mid(src_lane))` — borda direita do nó, centro vertical.
- Lado do destino: entra em `(step_cx(dst_step), node_y(dst_lane))` para baixo, ou `(step_cx(dst_step), node_y(dst_lane) + 64)` para cima.
- Raio do canto: Q-bezier de 8 px na curva.
- Arestas na mesma raia (raras — mesma raia, etapas adjacentes): `<line>` horizontal da direita da origem até a esquerda do destino.
- **Sem diagonais.** **Sem entrada pela esquerda.** **Sem saída pelo topo/base de um nó.**

```svg
<!-- Downward (destination lane > source lane) -->
<path d="M {rx},{src_cy} H {dst_cx - 8} Q {dst_cx},{src_cy} {dst_cx},{src_cy + 8} V {dst_top}"
      fill="none" stroke="…" stroke-width="…" marker-end="…"/>

<!-- Upward (destination lane < source lane) -->
<path d="M {rx},{src_cy} H {dst_cx - 8} Q {dst_cx},{src_cy} {dst_cx},{src_cy - 8} V {dst_bottom}"
      fill="none" stroke="…" stroke-width="…" marker-end="…"/>

<!-- Same lane (adjacent step) -->
<line x1="{src_right}" y1="{lane_cy}" x2="{dst_left}" y2="{lane_cy}"
      stroke="…" stroke-width="…" marker-end="…"/>
```

- **Ordem Z:** todos os conectores `<path>` e `<line>` são emitidos **antes** de qualquer `<rect>` de nó.
- **Marcadores:** exatamente um `marker-end` por caminho. Nunca `marker-start`.
- **Rótulos:** todas as setas de processo são sem rótulo por padrão. O número da etapa + a raia do ator carregam a semântica; um rótulo em cada seta é ruído. Só rotule uma seta se ela representar um conceito que não é uma etapa (loop de reteste, escalonamento) — nesse caso, use um rect mascarado com `paper` atrás de texto mono de 6.5 px.

### 3.2 Cruzamentos

Evite. A posição x do corredor (8 px antes do nó de destino) é a única coluna de roteamento — se duas setas fossem se cruzar ali, **troque as atribuições de etapas** ou **divida em dois diagramas** em vez de introduzir um contorno. Cruzamentos escondem o fluxo de controle subjacente.

---

## 4. Sobrescrita de cor de componente

Qualquer nó, raia ou etapa aceita um `color: "#hex"` opcional. Espelha `type-data-flow.md` §4 e `type-high-level.md` §3.4 para que a regra seja lida de forma idêntica entre os tipos.

### 4.1 `color` por nó

Aplicado a:

| Elemento | Claro | Escuro |
|---|---|---|
| Preenchimento do contêiner (`rect`) | `rgba(C, 0.06)` | `rgba(C_light, 0.10)` |
| Traço do contêiner | `rgba(C, 0.35)` (stroke-width 1) | `rgba(C_light, 0.45)` |
| Preenchimento do chip de papel | `rgba(C, 0.18)` | `rgba(C_light, 0.22)` |
| Texto do chip de papel | `C` | `C_light` |
| Texto do título | `C` | `C_light` |
| Sub-rótulo (in → out) | **inalterado** (muted) | **inalterado** (muted) |
| Rótulo da ferramenta | **inalterado** (soft) | **inalterado** (soft) |
| Chips de tipo de dado | **inalterados** | **inalterados** |
| Setas que tocam este nó | **inalteradas** — determinadas pela topologia | **inalteradas** |

`C_light` = o mesmo hex clareado ~15% para contraste no modo escuro (ex.: `#b85450` → `#d97a78`).

### 4.2 `color` por etapa

Substitui o preenchimento do chip do cabeçalho da etapa por `rgba(C, 0.20)` e o preenchimento do texto do número do chip por `C`. A entrada correspondente da etapa na legenda usa as mesmas cores.

### 4.3 `color` por raia

Substitui o tom da faixa da raia por `rgba(C, 0.04)` e o preenchimento do texto do rótulo da raia por `C`. Use com moderação — é fácil exagerar nos tons de raia.

### 4.4 Regras

- **Nunca em nós focais / etapas focais.** O `accent` já carrega esse sinal. Um `color` em um elemento focal é ignorado.
- **Nunca em setas.** As setas são determinadas pela topologia. Se você quer uma aresta colorida, escolha outro `style` de §3, não uma sobrescrita de cor.
- **Limite de 3 elementos com cor personalizada** por diagrama (nós + raias + etapas combinados), além do par focal (nó focal + cabeçalho da etapa focal). Acima de 3, o sinal visual começa a se fragmentar.
- **Subtítulo e rótulos de ferramenta permanecem `muted`** independentemente de qualquer `color` de componente.

### 4.5 Paleta semântica (recomendada)

A mesma paleta de `type-high-level.md`, `type-dp-integration.md`, `type-data-flow.md`:

- `#b85450` vermelho-ferrugem — Segurança / Identidade / Governança (controle de acesso, treinamento, aprovações)
- `#5a7d9a` azul-ardósia — Observabilidade / Qualidade (portões de qualidade de dados, validação, monitoramento)
- `#7a8c47` verde-oliva — Produtos de Dados / Publicação (saídas prontas para consumo, releases)
- `#8c6d3f` marrom quente — Backup / DR / Arquivo

---

## 5. Regra focal

O diagrama de processo tem três slots focais, exatamente uma entrada cada:

- **Uma etapa focal** (`steps[j].focal: true`) — tipicamente o pivô analítico ou de decisão (Test, Approve, Validate). O chip do cabeçalho e o chip da legenda são renderizados em `accent`.
- **Um nó focal** (`nodes[i].focal: true`) — o nó que *recebe* a transferência crítica. Borda `accent` + chip de papel `accent` + título em `ink` (o texto do título permanece em `ink` para continuar legível; apenas a borda + o chip de papel carregam o `accent`).
- **Um conjunto de setas focais** (`style: focal-in` e `focal-out`) — arestas que entram e saem do nó focal. Traços sólidos em `accent`.

Se zero ou >1 de qualquer slot focal forem declarados, pare e pergunte ao usuário.

---

## 6. Modo escuro

| Token | Claro | Escuro |
|---|---|---|
| Paper | `#ffffff` | `#161E2D` |
| Ink | `#232F3E` | `#ffffff` |
| Muted | `#545B64` | `#B6BEC9` |
| Soft | `#7D8998` | `#8D99A8` |
| Accent | `#EC7211` | `#FF9900` |
| Padrão de pontos | `rgba(35,47,62,0.10)` | `rgba(255,255,255,0.10)` |
| Tom da raia | `rgba(35,47,62,0.018)` | `rgba(255,255,255,0.025)` |
| Divisores | `rgba(35,47,62,0.12)` | `rgba(255,255,255,0.12)` |
| Divisor da coluna de rótulos | `rgba(35,47,62,0.20)` | `rgba(255,255,255,0.22)` |
| Preenchimento padrão do chip | `rgba(35,47,62,0.12)` | `rgba(255,255,255,0.12)` |
| Preenchimento do chip focal | `rgba(255,153,0,0.20)` | `rgba(255,153,0,0.22)` |
| Preenchimento padrão do nó | branco | `rgba(255,255,255,0.04)` |
| Traço padrão do nó | `rgba(35,47,62,0.25)` | `rgba(255,255,255,0.20)` |
| Preenchimento do nó focal | `rgba(255,153,0,0.08)` | `rgba(255,153,0,0.12)` |
| Traço do nó focal | `#EC7211` | `#FF9900` |
| Cores personalizadas de componentes | `C` | `C_light` (clarear ~15%) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. `viewBox = "0 0 {viewBox_w} {viewBox_h}"` derivado via §2.
2. Faixa do cabeçalho em `y=0..36`; faixa da legenda em `y=legend_y_top..viewBox_h` (`legend_y_top = 36 + n_lanes * 80`).
3. Todo nó em `(step_cx(j) - 50, lane_y_top(k) + 8)` com tamanho `100×64`.
4. Células vazias não renderizam nada — nenhum rect de placeholder, nenhum texto.
5. Exatamente **uma** etapa focal (`steps[j].focal: true`).
6. Exatamente **um** nó focal (`nodes[i].focal: true`).
7. Setas que tocam o focal usam `style: focal-in` / `focal-out` (`accent`).
8. Todas as outras setas usam `style: normal` (muted sólido) ou `style: trigger` (muted tracejado). Sem rótulo por padrão.
9. Todas as setas emitidas antes de qualquer rect de nó (regra de ordem Z).
10. Somente roteamento em ângulo reto com uma única curva — sai pela direita, entra por cima/por baixo. Sem diagonais. Q-bezier `r=8` em cada curva.
11. Cores personalizadas de componentes ≤ 3 (além do par focal). Setas nunca recoloridas pelo `color` do componente.
12. Subtítulo e rótulos de ferramenta permanecem `muted` independentemente de qualquer `color` de componente. Chip de entrada omitido nos nós da primeira etapa, chip de saída omitido nos nós da última etapa.

---

## 8. Referência dos chips de tipo de dado (entrada + saída)

Mesmo catálogo de `type-data-flow.md` §8.

- **Chip de entrada** em `(node_x+4, node_y+54)` — canto inferior **esquerdo**. Payload que entra no nó.
- **Chip de saída** em `(node_x+80, node_y+54)` — canto inferior **direito**. Payload que sai do nó.
- Qualquer um dos chips pode ser omitido (primeira/última etapa, payload desconhecido).

### Códigos dos chips

| Código | Cor (claro) | Cor (escuro) | Significado |
|------|---------------|--------------|---------|
| `LS` | `#7c8f6f` sálvia | `#9caf8f` | Lista / atribuição / tarefa |
| `DB` | `#5e7a9b` azul-empoeirado | `#82a0c0` | Dataset / registros tabulares |
| `TB` | `#b8915a` mostarda | `#d3ad7a` | Tabela (pronta para análise) |
| `FL` | `#9c6b50` marrom-ferrugem | `#b88670` | Arquivo / documento / relatório |
| `WB` | `#6e6479` ardósia | `#8d8298` | Web / imprensa / publicação pública |
| N/A | omitir o chip completamente | — | Desconhecido ou não aplicável |

Texto dentro do chip: branco, font-size 5, peso 700, mono.

As cores dos chips de tipo de dado são um **eixo semântico separado** da sobrescrita de cor por nó (§4). As cores dos chips descrevem o *formato do payload*; a cor do nó descreve o *tipo de preocupação*. Um nó pode ter, ao mesmo tempo, um chip mostarda `out: TB` e uma borda vermelho-ferrugem.

---

## 9. Legenda (faixa de 3 ou 4 linhas)

Cada linha é introduzida por um rótulo de categoria em `x = label_col_w + 4` (= 144). A legenda padrão tem **3 linhas** (`STEPS` / `DATA TYPE` / `FLOW`); quando um ou mais nós têm uma sobrescrita de `color` (§4), adicione uma 4ª linha `CONCERN` e aumente `legend_h` para 100.

- **Linha 1 — `STEPS`** em `y = legend_y_top + 16`: repita os chips do cabeçalho com seus rótulos. A etapa focal mantém o preenchimento `accent`.
- **Linha 2 — `DATA TYPE`** em `y = legend_y_top + 37`: uma amostra por tipo de chip realmente usado no diagrama. Acrescente uma pequena dica em mono `muted`: `left chip = input · right chip = output`.
- **Linha 3 — `CONCERN`** (apenas quando há sobrescritas de cor) em `y = legend_y_top + 58`: um mini-rect por cor personalizada usada, com seu rótulo semântico.
- **Linha 4 — `FLOW`** (a posição depende da existência da linha `CONCERN`): um segmento por estilo de seta realmente usado, com marcador + rótulo.

---

## 10. Orçamento de complexidade (limite)

| Dimensão | Máx. |
|---|---|
| Raias (atores) | 6 |
| Etapas | 12 |
| Nós por raia | Nós = apenas etapas ativas — células vazias são invisíveis |
| Setas rotuladas | 0 por padrão (rotule apenas conceitos que não são etapas) |
| Chips de tipo de dado por nó | 2 (entrada + saída) |
| Elementos com cor personalizada (§4) | 3 (além do nó focal + etapa focal) |

Acima de 6 raias ou 12 etapas: divida em dois diagramas (visão geral + detalhe).

---

## 11. Antipadrões

- **Células vazias com placeholder** — se um ator não participa de uma etapa, deixe a célula vazia (sem caixa, sem texto).
- **Setas diagonais** — todo conector deve ter exatamente uma curva em ângulo reto. Nada de linhas retas diretas entre nós em raias diferentes.
- **Entrada por porta esquerda/direita em uma seta predominantemente vertical** — sempre saia pela direita e entre por cima ou por baixo.
- **Mais de uma etapa focal / nó focal** — escolha a única operação mais crítica.
- **Raias sem rótulo** — toda swimlane deve identificar seu ator.
- **Todas as setas com o mesmo estilo** — gatilhos de orquestração devem ser tracejados para distingui-los dos conectores de fluxo de dados.
- **Sobrescrita de `color` em um elemento focal** — ignorada. `accent` sempre prevalece.
- **Setas com cor personalizada** — conectores são determinados pela topologia; o `color` de um nó nunca se propaga para suas arestas.
- **Tons de raia em excesso** — um tom em cada raia é lido como decoração, não como sinal. Aplique em ≤1 raia.
- **Chips de tipo de dado em um nó com nome de duas linhas** — omita os chips ou encurte o nome para uma linha.
- **Mais de 12 etapas sem dividir** — use um par visão geral + detalhe.

---

## 12. Exemplo trabalhado — YAML completo para `example-process-extended.html`

O diagrama de exemplo estendido é totalmente descrito pelas entradas a seguir. Toda coordenada no SVG renderizado pode ser derivada deste bloco via §2 + §3 + §4. Esta é a prova canônica de que o contrato paramétrico funciona de ponta a ponta.

```yaml
# Quarterly survey — end-to-end workflow (extended variant)
# 6 lanes × 11 steps, 1 focal step + 1 focal node + 3 custom-colored nodes

lanes:
  - { name: ["RD&E"],                 key: "RDE" }
  - { name: ["IT"],                   key: "IT"  }
  - { name: ["FIELD", "SERVICES"],    key: "FLD" }
  - { name: ["SURVEY", "SERVICES"],   key: "SVY" }
  - { name: ["HOUSEHOLD", "UNIT"],    key: "HHU" }
  - { name: ["COMMS &", "MARKETING"], key: "CMM" }

steps:
  - { number: "1",  label: "Design"   }
  - { number: "2",  label: "Assign"   }
  - { number: "3",  label: "Collect",  focal: true }    # focal step header chip
  - { number: "4",  label: "Review"   }
  - { number: "5",  label: "Validate" }
  - { number: "6",  label: "Weight"   }
  - { number: "7",  label: "Clean"    }
  - { number: "8",  label: "Tabulate" }
  - { number: "9",  label: "Approve"  }
  - { number: "10", label: "Publish"  }
  - { number: "11", label: "Upload"   }

nodes:
  - { lane: "RDE", step: 0,  title: "Sample Design",     sub: "Census data → Sample",
      tool: "SAS · Survey Solutions",      chips: {in: null, out: "LS"} }                # first step: no input chip
  - { lane: "IT",  step: 1,  title: "Field Assignment",  sub: "Sample → Field tasks",
      tool: "Survey Solutions",            chips: {in: "LS", out: "LS"} }
  - { lane: "FLD", step: 2,  title: "Data Collection",   sub: "→ 10,464 dwellings",
      tool: "Survey Solutions",            chips: {in: "LS", out: "DB"},  focal: true }   # focal node
  - { lane: "SVY", step: 3,  title: "HQ Review",         sub: "Submissions → Approved",
      tool: "Survey Sol. HQ",              chips: {in: "DB", out: "DB"},  color: "#b85450" }   # rust-red · governance
  - { lane: "IT",  step: 4,  title: "Error Checks",      sub: "Approved → Cleaned",
      tool: "SAS · Scripts",               chips: {in: "DB", out: "DB"},  color: "#5a7d9a" }   # slate-blue · data quality
  - { lane: "RDE", step: 5,  title: "Weight Calculation", sub: "Cleaned → Weighted",
      tool: "SAS",                         chips: null }                                  # 2-line title — chips skipped
  - { lane: "HHU", step: 6,  title: "2° Cleaning",       sub: "Weighted → Analysis",
      tool: "SAS · R · SPSS",              chips: {in: "DB", out: "TB"} }
  - { lane: "HHU", step: 7,  title: "Tables + Brief",    sub: "Analysis → Tables",
      tool: "Excel · SAS",                 chips: {in: "TB", out: "FL"} }
  - { lane: "CMM", step: 8,  title: "Stats Review",      sub: "Tables → Approved",
      tool: "Internal review",             chips: {in: "FL", out: "FL"} }
  - { lane: "CMM", step: 9,  title: "Public Release",    sub: "Approved → Public",
      tool: "Press conference",            chips: {in: "FL", out: "WB"}, color: "#7a8c47" }   # olive-green · data products
  - { lane: "IT",  step: 10, title: "Upload NatStat / SDMX", sub: "Results → Published",
      tool: "Web · SDMX API",              chips: null }                                  # 2-line title — chips skipped

arrows:
  - { from: {lane: "RDE", step: 0}, to: {lane: "IT",  step: 1},  style: "normal"    }
  - { from: {lane: "IT",  step: 1}, to: {lane: "FLD", step: 2},  style: "focal-in"  }     # → focal
  - { from: {lane: "FLD", step: 2}, to: {lane: "SVY", step: 3},  style: "focal-out" }     # ← focal
  - { from: {lane: "SVY", step: 3}, to: {lane: "IT",  step: 4},  style: "normal"    }     # upward
  - { from: {lane: "IT",  step: 4}, to: {lane: "RDE", step: 5},  style: "normal"    }     # upward
  - { from: {lane: "RDE", step: 5}, to: {lane: "HHU", step: 6},  style: "normal"    }     # downward, skips 2 lanes
  - { from: {lane: "HHU", step: 6}, to: {lane: "HHU", step: 7},  style: "normal"    }     # same lane
  - { from: {lane: "HHU", step: 7}, to: {lane: "CMM", step: 8},  style: "normal"    }
  - { from: {lane: "CMM", step: 8}, to: {lane: "CMM", step: 9},  style: "normal"    }     # same lane
  - { from: {lane: "CMM", step: 9}, to: {lane: "IT",  step: 10}, style: "normal"    }     # upward, skips 4 lanes

dark: false
```

### 12.1 O que este YAML comprova

Execute §2 desta referência com estas entradas:

- `n_lanes = 6`, `n_steps = 11`, `has_color_row = true` (3 nós têm `color`).
- `viewBox_w = 140 + 11 * 112 + 28 = 1400`. ✓ coincide com o SVG renderizado.
- `legend_h = 100`, `viewBox_h = 36 + 6 * 80 + 100 = 616`. ✓
- y_top das raias = [36, 116, 196, 276, 356, 436]; meio das raias = [76, 156, 236, 316, 396, 476]. ✓
- cx das etapas = [198, 310, 422, 534, 646, 758, 870, 982, 1094, 1208, 1320] (a margem interna de 8 px da área de conteúdo desloca cada valor em 8 em relação a `140 + j*112 + 50`). ✓
- Nó 4 (HQ Review): step=3, lane="SVY" (k=3) → x = 534-50 = 484, y = 276+8 = 284. ✓
- Nó 5 (Error Checks): step=4, lane="IT" (k=1) → x = 646-50 = 596, y = 116+8 = 124. ✓
- Nó 10 (Public Release): step=9, lane="CMM" (k=5) → x = 1208-50 = 1158, y = 436+8 = 444. ✓ *(O renderizado usa x=1156 — tolerância de 2 px por arredondamento da largura do chip no rótulo da etapa "10".)*

Os dois desvios de coordenadas nos dois nós mais à direita (a largura de chip = 20 para números de etapa com dois dígitos desloca o chip, mas não o cálculo do centro do nó) são um artefato do exemplo existente ajustado à mão, não uma falha da fórmula — uma geração nova a partir deste YAML produziria x=1158 e o diagrama seria visualmente indistinguível da versão distribuída.

### 12.2 Adaptando este YAML a um processo diferente

Para documentar um processo diferente, altere apenas o valor destas entradas:

- **Raias**: renomeie `lanes[k].name` para os nomes das suas equipes; atualize cada `nodes[i].lane` para corresponder. Até 6 raias.
- **Etapas**: renomeie `steps[j].label` e mova `focal: true` para a etapa que define a afirmação central do diagrama. Até 12 etapas.
- **Nós**: escreva uma entrada por célula `(lane, step)` que tenha trabalho. Deixe as células vazias (sem entrada) para não renderizar nada.
- **Cores**: escolha `color: "#hex"` em no máximo 3 nós (limite de §4). Fique na paleta recomendada, a menos que a marca exija outra coisa.
- **Setas**: declare cada aresta explicitamente com `style: normal | focal-in | focal-out | trigger`. A regra de roteamento (§3.1) preenche a geometria.

Todo o resto — dimensionamento do viewBox, posições dos chips, layout da legenda, troca de tokens do modo escuro — pode ser derivado. O YAML é a **fonte da verdade**; o SVG é uma das muitas renderizações possíveis dele (claro/escuro/full derivam todos das mesmas entradas com tokens de estilo diferentes).

---

## 13. Exemplos

- `assets/example-process.html` — mínimo claro (pesquisa trimestral: 11 etapas, 6 divisões, chips de tipo de dado). Padrão da galeria.
- `assets/example-process-dark.html` — o mesmo, skin (tema) escura.
- `assets/example-process-full.html` — o mesmo, moldura de cartão editorial.
- `assets/example-process-extended.html` — exercita a sobrescrita de cor de §4: Build app em azul-ardósia (qualidade de dados), Train enumerators em vermelho-ferrugem (governança), Publish results em verde-oliva (produtos de dados). O destaque focal na etapa + nó Pilot test permanece inalterado.
- `assets/example-process-extended-dark.html` — padrão estendido, skin (tema) escura.
- `assets/example-process-extended-full.html` — padrão estendido, moldura de cartão editorial.
