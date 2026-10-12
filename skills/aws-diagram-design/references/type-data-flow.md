# Fluxo de dados (Data Flow)

**Ideal para:** visualizar como os dados se movem por um pipeline *entre papéis organizacionais* — quem inicia, quem processa, quem publica e quem consome. O caso de uso canônico é uma plataforma de dados com múltiplos papéis (Admin → Engineers → Scientists → Consumers) com 4–6 etapas de processo. Use quando o leitor precisa entender **quem faz o quê em cada estágio**, e não apenas os componentes técnicos.

Prefira o **Swimlane** padrão para processos de negócio multifuncionais (aprovações de RH, tickets de suporte). Use **Data flow** quando o assunto for um pipeline de dados com payloads tipados (arquivos brutos, tabelas, relatórios) e fronteiras de acesso delimitadas por papel.

Este tipo é **paramétrico** — o esquema de entradas da §1 determina cada coordenada por meio das fórmulas da §2. Duas gerações a partir das mesmas entradas devem produzir SVGs visualmente idênticos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
lanes:                              # 1..4 horizontal swimlanes (top to bottom)
  - { name: ["DATA", "ADMINS"],     key: "ADM" }
  - { name: ["DATA", "ENGINEERS"],  key: "ENG" }
  - { name: ["DATA", "SCIENTISTS"], key: "SCI" }
  - { name: ["DATA", "CONSUMERS"],  key: "CON" }

steps:                              # 1..6 columns (left to right)
  - { number: "01", label: "COLLECT" }
  - { number: "02", label: "STORE" }
  - { number: "03", label: "TRANSFORM" }
  - { number: "04", label: "ANALYZE",  focal: true }   # focal step header chip — accent fill
  - { number: "05", label: "PUBLISH" }

nodes:                              # explicit per-cell entries; empty cells render nothing
  - { lane: "ADM", step: 0, title: "Project Setup",   sub: "create · assign roles",     tool: "Platform console" }
  - { lane: "ADM", step: 1, title: "Access Control",  sub: "bucket policies · LDAP",    tool: "MinIO · LDAP console",
      color: "#b85450" }            # tinted rust-red to flag governance/identity concern
  - { lane: "ENG", step: 0, title: "Source Ingest",   sub: "ext. sources → raw",        tool: "NiFi · API · SFTP",
      chips: {in: "WB", out: "DB"} }                    # web payload in, dataset out
  - { lane: "ENG", step: 1, title: "Raw Store",       sub: "raw landing zones",         tool: "MinIO raw",
      chips: {in: "DB", out: "DB"} }                    # raw stays raw inside the landing zone
  - { lane: "ENG", step: 2, title: "Clean & Stage",   sub: "raw → staging → anon",      tool: "NiFi · Trino",
      chips: {in: "DB", out: "TB"} }                    # raw dataset → analysis-ready table
  - { lane: "SCI", step: 3, title: "Explore & Model", sub: "anon data → insights",      tool: "JupyterHub · Trino",
      chips: {in: "TB", out: "FL"}, focal: true }       # focal — table in, file/report out
  - { lane: "SCI", step: 4, title: "Publish Findings", sub: "models → dashboards",      tool: "Superset · Reports",
      chips: {in: "FL", out: "FL"} }                    # report in, report out (pass-through to publish)
  - { lane: "CON", step: 4, title: "Query Insights",   sub: "aggregated views",         tool: "Trino (read-only)",
      chips: {in: "TB", out: "TB"} }                    # consumers read tables, hand off tables

arrows:                             # explicit edges; styles bind to topology (see §3)
  - { from: {lane: "ADM", step: 0}, to: {lane: "ADM", step: 1}, style: "muted" }
  - { from: {lane: "ADM", step: 0}, to: {lane: "ENG", step: 0}, style: "trigger" }   # dashed governance
  - { from: {lane: "ADM", step: 1}, to: {lane: "ENG", step: 1}, style: "trigger" }
  - { from: {lane: "ENG", step: 0}, to: {lane: "ENG", step: 1}, style: "muted" }
  - { from: {lane: "ENG", step: 1}, to: {lane: "ENG", step: 2}, style: "muted" }
  - { from: {lane: "ENG", step: 2}, to: {lane: "SCI", step: 3}, style: "accent",     # focal cross-role
      label: "anon data" }
  - { from: {lane: "SCI", step: 3}, to: {lane: "SCI", step: 4}, style: "muted" }
  - { from: {lane: "SCI", step: 4}, to: {lane: "CON", step: 4}, style: "link" }     # teal: published

dark: false
```

**Semântica dos campos reservados:**
- `lanes[k].key` — o texto de 3 letras do chip de papel (ex.: `ADM`, `ENG`, `SCI`, `CON`). Usado dentro de cada nó daquela raia.
- `lanes[k].name` — rótulo da raia em duas linhas; ambas as linhas usam o papel `eyebrow` em maiúsculas.
- `steps[j].focal: true` — exatamente **uma** etapa pode declarar isso. O chip do cabeçalho é renderizado em accent.
- `nodes[i].focal: true` — exatamente **um** nó pode declarar isso. É renderizado com borda accent (§5).
- `nodes[i].chips` — chips de tipo de dado do nó. Qualquer uma das formas é aceita:
  - **Forma de objeto (preferida):** `{in: "<CODE>", out: "<CODE>"}` — semântica explícita de entrada/saída. Qualquer um dos lados é opcional.
  - **Forma de array:** `["<INPUT_CODE>", "<OUTPUT_CODE>"]` — o primeiro item é a entrada, o segundo é a saída.
  - Códigos da §8 (`WB`, `DB`, `TB`, `FL`, `LS`). A posição é **fixa**: chip de entrada no canto inferior **esquerdo** do nó, chip de saída no canto inferior **direito**.
- `nodes[i].color` — **substituição de cor por nó** opcional. Qualquer string `"#hex"` válida é aceita; a paleta da §4 é recomendada para consistência entre diagramas, mas não é obrigatória. Cada nó pode ter sua própria cor, independentemente dos demais.
- `nodes[i].aws_icon` — ícone oficial do AWS Architecture Icons para o serviço principal do nó (caminho em `assets/aws-icons/`). **Obrigatório sempre que `tool` nomear um serviço AWS específico** (instalação com skin AWS) — veja [primitive-aws-icons.md § Nós compactos (Compact nodes)](primitive-aws-icons.md). Renderizado no slot de 16px do canto superior direito (§2.4); um ícone por nó, o do serviço dominante quando `tool` listar vários.

---

## 2. Fórmulas de layout — geometria determinística

```
label_col_w      = 140
step_slot_w      = 112
right_pad        = 28
n_steps          = len(steps)
n_lanes          = len(lanes)

# Canvas
viewBox_w        = label_col_w + n_steps * step_slot_w + right_pad   # 5 steps → 728
header_h         = 36
lane_h           = 80
has_color_row    = any(node.color or step.color or lane.color in inputs)
legend_h         = 100 if has_color_row else 80                      # 4 rows when colors are present
viewBox_h        = header_h + n_lanes * lane_h + legend_h            # 4 lanes, no colors → 436; with colors → 456

# Header strip (top)
header_y         = 0                                                  # ends at header_h = 36
step_chip_y      = 6                                                  # 16-px chip at y=6..22
step_label_y     = 29                                                 # text line below chip

# Lane positions
lane_y_top(k)    = header_h + k * lane_h                              # 36, 116, 196, 276
lane_y_mid(k)    = lane_y_top(k) + lane_h/2                           # 76, 156, 236, 316
lane_label_x     = label_col_w / 2                                    # 70

# Step / node center x
step_cx(j)       = label_col_w + j * step_slot_w + step_slot_w/2      # 196, 308, 420, 532, 644

# Nodes
node_w           = 100
node_h           = 64
node_x(j)        = step_cx(j) - node_w/2                              # 146, 258, 370, 482, 594
node_y(k)        = lane_y_top(k) + 8                                  # 44, 124, 204, 284

# Legend strip (bottom)
legend_y_top     = header_h + n_lanes * lane_h                        # 356
legend_row_y     = [legend_y_top + 16, legend_y_top + 37, legend_y_top + 59]
                                                                      # 372, 393, 415
```

### 2.1 Estrutura de fundo

- Fill de paper em todo o viewBox.
- Padrão de pontos: grade de 22×22, `circle r=0.8`, `fill ink @ 0.10`.
- Tons alternados nas raias: raias de índice ímpar (0, 2, …) recebem fill `ink @ 0.018`.
- Divisores de raia: linhas finas (hairlines) horizontais em cada `lane_y_top(k)` e em `legend_y_top`, stroke `ink @ 0.12` com largura 0.8.
- Borda direita da coluna de rótulos: linha fina vertical em `x = label_col_w`, de `y = header_h` até `y = legend_y_top`.

### 2.2 Chip do cabeçalho da etapa

Para cada etapa `j`:

```
chip_x(j)        = step_cx(j) - 16        # 16×16 chip
chip_y           = 6
chip_w           = 32
chip_h           = 16
chip_rx          = 8                       # pill-shaped
number_anchor    = (step_cx(j), 14)
label_anchor     = (step_cx(j), 29)
```

Fill padrão: `ink @ 0.12`, texto do número em ink, texto do rótulo em muted.
Fill focal: `accent @ 0.20`, texto do número + rótulo em accent.
Substituição de `color` por etapa (§4): substitui o fill por `rgba(C, 0.20)` e o fill do texto por `C`.

### 2.3 Rótulos das raias

Rótulo de duas linhas com o papel `eyebrow`, ambas as linhas em maiúsculas, fill muted:
- Linha 1 em `(lane_label_x, lane_y_mid(k) - 4)`
- Linha 2 em `(lane_label_x, lane_y_mid(k) + 8)`

Substituição de `color` por raia (§4): substitui o fill do rótulo por `C` e o tom da raia por `rgba(C, 0.04)` (em vez do padrão `ink @ 0.018`).

### 2.4 Layout do conteúdo do nó (dentro do rect de 100×64)

```
role_chip          rect 18×10 at (node_x+4, node_y+4),  rx=3
aws_icon           16×16 at (node_x+80, node_y+4)                  # top-right corner slot, scale 16/64=0.25
                                                                   # only when nodes[i].aws_icon is set
title              centered at (step_cx(j), node_y+23), node-name role, font-size=9
sub                centered at (step_cx(j), node_y+35), sublabel role, font-size=6.5, muted
tool               centered at (step_cx(j), node_y+47), sublabel role, font-size=6.5, soft
data chip IN       rect 16×8 at (node_x+4,   node_y+54), rx=3      # payload type entering the node
data chip OUT      rect 16×8 at (node_x+80,  node_y+54), rx=3      # payload type leaving the node
role_chip_text     centered at (node_x+13, node_y+9), eyebrow role, font-size=6, weight=600
```

Quando `aws_icon` estiver presente, limite o título a **~13 caracteres** para que o título centralizado não colida com o ícone do canto (veja [primitive-aws-icons.md § Nós compactos (Compact nodes)](primitive-aws-icons.md)).

Células vazias (sem entrada de nó) não renderizam **nada**. Nenhum rect de placeholder, nenhum chip de papel, nenhum rótulo — a célula fica invisível.

---

## 3. Regras de setas (obrigatórias)

Quatro estilos, vinculados à topologia. Os conectores são desenhados **antes** de todos os rects de nó (regra de ordem Z).

| `style` | Traço | Largura | Tracejado | Marcador | Quando é obrigatório |
|---|---|---|---|---|---|
| `muted` | `muted` | 1.0 | — | `arr-muted` | Passagem de dados padrão entre etapas ou dentro de uma raia. |
| `trigger` | `muted` | 1.0 | `4,3` | `arr-muted` | Gatilho de governança — uma ação de admin habilita o trabalho a jusante. Sem rótulo. |
| `accent` | `accent` | 1.2 | — | `arr-accent` | Passagem focal entre papéis. **Exatamente uma por diagrama**, com rótulo. |
| `link` | `link` | 1.0 | — | `arr-link` | Saída publicada / consumida externamente. |

**Bloco defs** (obrigatório, três marcadores):

```svg
<defs>
  <pattern id="dots" width="22" height="22" patternUnits="userSpaceOnUse">
    <circle cx="11" cy="11" r="0.8" fill="{ink @ 0.10}"/>
  </pattern>
  <marker id="arr-muted"  markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="{muted}"/></marker>
  <marker id="arr-accent" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="{accent}"/></marker>
  <marker id="arr-link"   markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto"><path d="M0,0 L6,3 L0,6 Z" fill="{link}"/></marker>
</defs>
```

### 3.1 Regras de roteamento (inegociáveis)

- **Roteamento com uma única dobra:** primeiro horizontal, depois vertical. Saia de um nó pela **borda direita**; entre pela **esquerda** (horizontal na mesma raia) ou pelo **topo/base** (vertical entre raias).
- **Sem diagonais.** As dobras usam um canto Q-bezier de 8 px.
- **Mesma etapa, entre raias (vertical)**: linha direta entre `(step_cx(j), lane_y_top(k_to)−12)` e `(step_cx(j), lane_y_top(k_to))`. Usada para gatilhos admin → engineers sob a mesma etapa.
- **Entre raias e entre etapas (focal)**: saia pela direita, siga na horizontal além da borda direita do nó de origem até um x de corredor logo antes da etapa do destino e então desça verticalmente.
- **Rótulos:** apenas a seta `accent` recebe rótulo. Use uma máscara de rect preenchida com paper (opaca) 6 px atrás do texto. As demais setas ficam sem rótulo.
- **Ordem Z:** todas as setas são emitidas antes de qualquer rect de nó (os fills dos rects mascaram as pontas das linhas dentro do nó).

---

## 4. Substituição de cor de componente

Qualquer nó, raia ou etapa pode declarar um `color: "#hex"` opcional. Espelha a §3.4 do high-level e a §4 do dp-integration para que a regra seja lida de forma idêntica entre os tipos.

### 4.1 `color` por nó

Aplicado a:

| Elemento | Claro | Escuro |
|---|---|---|
| Fill do contêiner (`rect`) | `rgba(C, 0.06)` | `rgba(C_light, 0.10)` |
| Stroke do contêiner | `rgba(C, 0.35)` (stroke-width 1) | `rgba(C_light, 0.45)` |
| Fill do chip de papel | `rgba(C, 0.18)` | `rgba(C_light, 0.22)` |
| Texto do chip de papel | `C` | `C_light` |
| Texto do título | `C` | `C_light` |
| Sub-rótulo | **inalterado** (muted) | **inalterado** (muted) |
| Rótulo da ferramenta | **inalterado** (soft) | **inalterado** (soft) |
| Chips de tipo de dado | **inalterados** | **inalterados** |
| Setas que tocam este nó | **inalteradas** — definidas pela topologia | **inalteradas** |

`C_light` = o mesmo hex clareado em ~15% para contraste no modo escuro (ex.: `#b85450` → `#d97a78`).

### 4.2 `color` por etapa

Substitui o fill do chip do cabeçalho da etapa por `rgba(C, 0.20)` e o fill do texto de número + rótulo do chip por `C`. A entrada correspondente da etapa na legenda usa as mesmas cores.

### 4.3 `color` por raia

Substitui o tom da faixa da raia por `rgba(C, 0.04)` (apenas para raias de índice ímpar que recebem tom por padrão — ou estenda a todas as raias se escolhido explicitamente) e o fill do texto do rótulo da raia por `C`. Use com moderação; tons de raia são fáceis de aplicar em excesso.

### 4.4 Regras

- **Nunca em nós focais / etapas focais.** O accent já carrega esse sinal. Um `color` em um elemento focal é ignorado.
- **Nunca em setas.** As setas são definidas pela topologia. Se você quer uma aresta colorida, escolha outro `style` da §3, não uma substituição de cor.
- **Limite de 3 elementos com cor personalizada** por diagrama (nós + raias + etapas somados), além do par focal (nó focal + cabeçalho da etapa focal). Acima de 3, o sinal visual começa a se fragmentar — se precisar de mais, divida o diagrama ou repense se cada cor carrega um significado distinto.
- **Subtítulos e rótulos de ferramenta permanecem muted.** Apenas a identidade principal (borda + chip de papel + título + ícone) carrega o sinal de cor.

### 4.5 Paleta semântica (recomendada)

A mesma paleta do high-level / dp-integration, para que um leitor que percorra vários diagramas veja as mesmas cores com o mesmo significado:

- `#b85450` vermelho-ferrugem — Segurança / Identidade / Governança (nós de admin, LDAP, controle de acesso)
- `#5a7d9a` azul-ardósia — Observabilidade / Qualidade (monitoramento, portões de qualidade de dados, linhagem)
- `#7a8c47` verde-oliva — Governança / Linhagem (catálogo, metadados)
- `#8c6d3f` marrom-quente — Backup / DR / Arquivamento

---

## 5. Regra focal

O diagrama de fluxo de dados é construído em torno de **uma passagem entre papéis** que define sua afirmação central. Três slots focais, exatamente uma entrada em cada:

- **Uma etapa focal** (`steps[j].focal: true`) — normalmente o pivô analítico (Analyze, Model, …). O chip do cabeçalho e o chip da legenda são renderizados em accent.
- **Um nó focal** (`nodes[i].focal: true`) — o nó que *recebe* a passagem focal. Borda accent + chip de papel accent + título em ink.
- **Uma seta focal** (`arrows[i].style: "accent"`) — a passagem entre papéis que chega ao nó focal. Traço accent sólido + rótulo com um descritor curto do payload (ex.: `anon data`).

Se zero ou mais de um de qualquer slot focal forem declarados, interrompa e pergunte ao usuário.

---

## 6. Modo escuro

| Token | Claro | Escuro |
|---|---|---|
| Paper | `paper` | `ink` |
| Ink | `ink` | `paper` |
| Muted | `muted` | `soft` |
| Soft | `soft` | `rule-solid` |
| Accent | `accent` | `accent` |
| Link | `link` | `link` |
| Padrão de pontos | `ink @ 0.10` | `paper @ 0.10` |
| Tom da raia | `ink @ 0.018` | `paper @ 0.025` |
| Divisores | `ink @ 0.12` | `paper @ 0.12` |
| Fill padrão do chip | `ink @ 0.12` | `paper @ 0.12` |
| Fill do chip focal | `accent @ 0.20` | `accent @ 0.22` |
| Fill padrão do nó | `paper` | `paper @ 0.04` |
| Stroke padrão do nó | `ink @ 0.25` | `paper @ 0.20` |
| Fill do nó focal | `accent @ 0.07` | `accent @ 0.12` |
| Stroke do nó focal | `accent` | `accent` |
| Cores personalizadas de componente | `C` | `C_light` (clarear ~15%) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. `viewBox = "0 0 {viewBox_w} {viewBox_h}"` derivado de `n_steps` e `n_lanes` pela §2.
2. Faixa do cabeçalho em `y=0..36`; faixa da legenda em `y=legend_y_top..viewBox_h` (`legend_y_top = 36 + n_lanes * 80`).
3. Cada nó em `(step_cx(j) - 50, lane_y_top(k) + 8)` com tamanho `100×64`.
4. Células vazias não renderizam nada — nenhum rect de placeholder, nenhum texto.
5. Exatamente **uma** etapa focal (`steps[j].focal: true`).
6. Exatamente **um** nó focal (`nodes[i].focal: true`).
7. Exatamente **uma** seta focal (`style: accent`). Com rótulo e um rect mascarado com paper atrás do rótulo.
8. Todas as demais setas sem rótulo.
9. Todas as setas emitidas antes de qualquer rect de nó (regra de ordem Z).
10. Apenas roteamento com uma única dobra — sem diagonais. Q-bezier `r=8` em cada dobra.
11. Cores personalizadas de componente ≤ 3 (além do par focal). Setas nunca são recoloridas pelo `color` do componente.
12. Subtítulos e rótulos de ferramenta permanecem muted, independentemente de qualquer `color` de componente.

---

## 8. Referência dos chips de tipo de dado (entrada + saída)

Pequenos badges `16×8 rx=3` na parte inferior de cada nó, um para entrada e outro para saída. A posição é **inegociável**:

- **Chip de entrada** em `(node_x+4, node_y+54)` — canto inferior **esquerdo** do nó. Representa o formato do payload que *entra* no nó vindo de montante.
- **Chip de saída** em `(node_x+80, node_y+54)` — canto inferior **direito** do nó. Representa o formato do payload que *sai* do nó em direção a jusante.

Qualquer um dos chips pode ser omitido (ex.: um nó de destino final tem apenas o chip de entrada; um nó apenas de origem tem apenas o chip de saída). A leitura do diagrama se torna um rastreamento da transformação do payload: percorra uma linha, leia a entrada → saída de cada nó e você verá exatamente qual forma os dados assumem em cada passagem.

### Códigos dos chips

| Código | Cor | Significado |
|------|-------|---------|
| `WB` | `#6e6479` (malva) | Dados web / públicos |
| `DB` | `#5e7a9b` (azul-aço) | Dataset / arquivo bruto |
| `TB` | `#b8915a` (âmbar) | Tabela / pronta para análise |
| `FL` | `#9c6b50` (siena) | Arquivo / relatório / exportação |
| `LS` | `#4a7c59` (verde-floresta) | Stream ao vivo / evento |

Texto dentro do chip: branco, papel `eyebrow` a 5px, peso 700.

As cores dos chips de tipo de dado são um **eixo semântico separado** da substituição de cor por nó (§4). As cores dos chips descrevem o *formato do payload*; a substituição de cor do nó descreve o *tipo de preocupação* (governança, observabilidade, …). Não as confunda — um nó pode ter, ao mesmo tempo, um chip âmbar `out: TB` e uma cor de borda vermelho-ferrugem.

---

## 9. Legenda (faixa de 3 ou 4 linhas)

Cada linha é introduzida por um rótulo de categoria em `x=144`. A legenda padrão tem **3 linhas** (`STEPS` / `DATA TYPE` / `FLOW`); quando um ou mais nós têm substituição de `color` (§4), adicione uma 4ª linha `CONCERN` e aumente `legend_h` para 100 (de modo que `viewBox_h = header_h + n_lanes·80 + 100`).

- **Linha 1 — `STEPS`** em `y = legend_y_top + 16`: repita os chips do cabeçalho com seus rótulos. A etapa focal mantém o fill accent.
- **Linha 2 — `DATA TYPE`** em `y = legend_y_top + 37`: uma amostra por tipo de chip efetivamente usado no diagrama. **Acrescente uma pequena dica** no papel `sublabel` em muted após os chips: `left chip = input · right chip = output`. Isso torna explícita, para quem lê pela primeira vez, a convenção de entrada/saída baseada em posição.
- **Linha 3 — `CONCERN`** (apenas quando houver substituições de cor) em `y = legend_y_top + 58`: um mini-rect por cor personalizada usada, com seu rótulo semântico (ex.: `Identity · Governance`, `Data Quality · Observability`). A amostra do destaque focal também é mostrada aqui, para que o leitor veja os três eixos coloridos lado a lado.
- **Linha 4 — `FLOW`** (última linha; a posição depende da existência da linha `CONCERN`): segmentos curtos de linha com seu marcador + rótulo de texto, um por estilo de seta efetivamente usado.

Todos os itens da legenda se alinham em uma única faixa horizontal por linha. Não empilhe verticalmente dentro de uma caixa.

---

## 10. Orçamento de complexidade

| Dimensão | Máx. |
|---|---|
| Raias (papéis) | 4 |
| Etapas | 6 |
| Nós por raia | Nós = apenas etapas ativas — células vazias são invisíveis (sem caixa de placeholder) |
| Setas com rótulo | 1 (apenas a accent focal) |
| Chips de tipo de dado por nó | 2 |
| Elementos com cor personalizada (§4) | 3 (além do nó focal + etapa focal) |

Acima de 4 raias ou 6 etapas: divida em dois diagramas (ex.: pipeline de ingestão / pipeline de análise).

---

## 11. Antipadrões

- **Células vazias com placeholder** — se um papel não participa de uma etapa, deixe a célula vazia (sem caixa, sem texto).
- **Mais de uma seta com rótulo** — apenas a passagem focal entre papéis recebe rótulo.
- **Setas diagonais** — sempre primeiro horizontal, depois vertical; uma única dobra em ângulo reto.
- **Papel `title` para títulos de nó** — títulos de nó usam o papel `node-name` (contexto técnico). Apenas o `<h1>` da página usa `title`.
- **Accent em mais de um nó, uma etapa, uma seta** — focal = um nó + uma etapa + uma seta, no máximo.
- **Papel `node-name` para rótulos de papel** — rótulos de raia sempre usam o papel `eyebrow` em maiúsculas (são identificadores, não prosa).
- **Substituição de `color` em um elemento focal** — ignorada. O accent sempre prevalece.
- **Setas com cor personalizada** — setas são definidas pela topologia. A cor de um nó nunca se propaga para suas arestas.
- **Tons de raia aplicados em excesso** — um tom em cada raia é lido como decoração, não como sinal. Aplique a ≤1 raia.

---

## 12. Exemplos

- `assets/example-data-flow.html` — mínimo claro (a plataforma, 4 papéis × 5 etapas: Admin, Engineers, Scientists, Consumers). Padrão da galeria.
- `assets/example-data-flow-dark.html` — o mesmo, skin (tema) escuro.
- `assets/example-data-flow-full.html` — o mesmo, moldura de cartão editorial.
- `assets/example-data-flow-extended.html` — exercita a substituição de cor da §4: nó Access Control em vermelho-ferrugem (governança), nó Clean & Stage em azul-ardósia (qualidade de dados). Destaque focal na etapa Analyze + nó Explore & Model + seta anon-data inalterado.
- `assets/example-data-flow-extended-dark.html` — padrão estendido, skin (tema) escuro.
- `assets/example-data-flow-extended-full.html` — padrão estendido, moldura de cartão editorial.
