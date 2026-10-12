# High-Level (visão de alto nível)

**Ideal para:** visões gerais de ponta a ponta de uma stack de dados — ingestão → armazenamento → consulta → análise → visualização — implantada em um orquestrador de contêineres (Kubernetes, ECS, Nomad). Combina uma faixa de chevrons de fases, uma fronteira de implantação, uma barra de orquestração, um rodapé de identidade e (opcionalmente) uma faixa vertical de chevrons à direita para preocupações transversais (Orchestration, Security, Observability).

Este tipo é **paramétrico**. O diagrama é totalmente determinado por uma pequena lista de entradas (chevrons, fontes, componentes, conexões). As fórmulas abaixo dizem exatamente onde cada forma fica a partir dessas entradas — duas gerações a partir das mesmas entradas devem produzir SVGs visualmente idênticos.

---

## 1. Entradas — o contrato de parâmetros

Antes de desenhar, colete estes dados do usuário (ou aceite-os como um bloco YAML/JSON). Tudo nesta referência é derivado dessas entradas. Não invente geometria na hora.

```yaml
chevrons:                       # ordered left → right; reserved names auto-promote to vertical
  - { name: "Data sources",         columns: 1 }
  - { name: "Ingestion",            columns: 1 }
  - { name: "Storage",              columns: 1 }
  - { name: "Transformation",       columns: 1 }
  - { name: "Visualization",        columns: 1 }
  - { name: "Orchestration",        vertical: true }                     # reserved → pairs with the bar
  - { name: "Security",             vertical: true, color: "#b85450" }   # tinted to match the Identity bar below
  - { name: "Observability",        vertical: true }                     # reserved → pairs with crosscut #2

sources:                        # external; rendered in the dashed zone on the left
  - { name: "PostgreSQL", type: "db",     connects_to: ["NiFi"] }
  - { name: "SFTP drop",  type: "ftp",    connects_to: ["NiFi"] }
  - { name: "Web forms",  type: "web",    connects_to: ["NiFi"] }
  - { name: "Legacy",     type: "legacy", connects_to: ["NiFi"] }

components:                     # inside the cluster, plus bars and cross-cutting rows
  - { name: "NiFi",       chevron: "Ingestion",      kind: node,          icon: nifi,     role: "COLL"  }
  - { name: "MinIO",      chevron: "Storage",        kind: node,          icon: minio,    role: "STORE", focal: true }
  - { name: "Trino",      chevron: "Storage",        kind: node,          icon: trino,    role: "VIRT"  }
  - { name: "Notebooks",  chevron: "Transformation", kind: node,          icon: jupyter,  role: "ANLZ"  }
  - { name: "Superset",   chevron: "Visualization",  kind: node,          icon: superset, role: "DASH" }
  - { name: "Airflow",    chevron: "Orchestration",  kind: bar,           icon: airflow,    subtitle: "Apache Airflow" }
  - { name: "Identity",   chevron: "Security",       kind: cross-cutting, icon: keycloak,   subtitle: "Keycloak · LDAP · OIDC",     color: "#b85450" }
  - { name: "Monitoring", chevron: "Observability",  kind: cross-cutting, icon: prometheus, subtitle: "Prometheus · Grafana · Loki" }

connections:                    # explicit edges; focal-touching ones become accent automatically
  - { from: "NiFi",    to: "MinIO",                style: "primary" }
  - { from: "NiFi",    to: "Trino",                style: "secondary" }
  - { from: "MinIO",   to: "Notebooks",            style: "primary" }
  - { from: "Trino",   to: "MinIO",                style: "query" }   # read-back (dashed)
  - { from: "Notebooks", to: "Superset",           style: "secondary" }
  - { from: "Airflow", to: ["NiFi", "Trino", "Notebooks"], style: "trigger" }

focal: "MinIO"                  # exactly one; defaults to first kind=node under "Storage"
dark: false
```

**Nomes de chevron reservados** (sempre verticais, mesmo se `vertical: true` for omitido): `Orchestration`, `Security`, `Observability`, `Governance`, `Backup`.

**Valores reservados de `kind`:**
- `node` — uma caixa padrão dentro do cluster (padrão).
- `bar` — uma faixa horizontal que ocupa o topo do cluster. Normalmente uma só (Orchestration); veja a regra de pareamento em §5.
- `cross-cutting` — uma faixa horizontal que ocupa a largura do corpo (termina na margem da faixa lateral), empilhada abaixo do cluster. **Zero ou mais permitidas**; cada uma é empilhada 44 px abaixo da anterior (§2.5) e forma par 1:1 com um chevron vertical (§5).

**`color` opcional** (por componente, string hex): colore o contêiner e o conteúdo do componente, sem alterar os conectores. Veja §3.4. Use com moderação — uma cor personalizada é um sinalizador semântico (ex.: vermelho = preocupação de segurança), não decoração.

**Valores de `type` de fonte** → mapeamento de ícones (use `references/primitive-icons.md`):
- `db` → `database`
- `ftp` → `bucket` ou seta de upload
- `web` → `internet`
- `legacy` → `server`
- `api` → `api`
- Qualquer nome de ícone explícito em `primitive-icons.md` também é aceito.

---

## 2. Fórmulas de layout — geometria determinística

Toda coordenada abaixo é derivada das entradas. **Nenhum número fixo nos exemplos que não esteja justificado aqui.**

### 2.1 Canvas

```
has_vertical       = any(c.vertical or c.name in reserved_names for c in chevrons)
right_strip_w      = 28  if has_vertical else 0
strip_margin       = 8   if has_vertical else 0   # gap between body and right strip
effective_w        = 1000 - right_strip_w - strip_margin     # 1000 or 964

n_cross            = count of components with kind == "cross-cutting"
strip_y_bot        = max(428, 388 + n_cross * 44 - 4)        # extends to last crosscut row
viewBox_h          = max(540, strip_y_bot + 112)             # 112 reserved for legend
viewBox            = f"0 0 1000 {viewBox_h}"
```

Todo elemento horizontal (faixa de chevrons, cluster, barra de orquestração, barras de identidade / transversais) termina em `effective_w`. A faixa da direita fica em `x = 1000 - right_strip_w` (= 972). A banda de 8 px entre eles é respiro visual — nunca coloque conteúdo ali.

`viewBox_h` cresce quando mais de uma barra transversal é declarada: 1 transversal → 540, 2 → 600 (ou 584 se quiser mais justo; a regra arredonda para o próximo múltiplo de 20 para grades limpas).

### 2.2 Faixa horizontal de chevrons

```
y_banner           = 4
h_banner           = 28
horizontals        = [c for c in chevrons if not c.vertical and c.name not in reserved_names]
sum_columns        = sum(c.columns for c in horizontals)
base_unit          = floor_to_4(effective_w / sum_columns)        # multiple of 4
widths             = [max(120, base_unit * c.columns) for c in horizontals]
widths[-1]         += effective_w - sum(widths)                   # trailing absorbs remainder
x_boundaries       = cumulative_sum([0] + widths)                 # length sum_columns+1
chevron_cx(C)      = (x_boundaries[index(C)] + x_boundaries[index(C)+1]) / 2
```

**Formas dos polígonos:**
- Primeiro (mais à esquerda): `(x0,4) (x1-12,4) (x1,18) (x1-12,32) (x0,32)`
- Do meio: `(x0,4) (x1-12,4) (x1,18) (x1-12,32) (x0,32) (x0+12,18)`
- Último (mais à direita): `(x0,4) (effective_w,4) (effective_w,32) (x0,32) (x0+12,18)`

Os preenchimentos alternam `#232F3E` / `#3d4460` (modo claro) ou `#3d4460` / `#4a5270` (modo escuro). Rótulos: mono na cor `paper`, `font-size=7`, `letter-spacing=0.14em`, `text-anchor=middle`, centralizados em `chevron_cx, 21`.

**Substituição de cor** (por chevron, tanto horizontal quanto vertical): um chevron pode declarar um `color: "#hex"` opcional que substitui o preenchimento da alternância apenas para aquele chevron. Use-o para sinalizar uma fase que forma par com um componente de cor personalizada (ex.: chevron `Security` em vermelho quando a barra Identity usa `color: "#b85450"`). Regras:

- A substituição se aplica apenas ao preenchimento do polígono. O rótulo continua na cor `paper` — nunca recolora rótulos de chevron.
- O índice de alternância não se desloca; os chevrons vizinhos mantêm seu preenchimento natural, mesmo que isso produza dois chevrons adjacentes com o mesmo preenchimento. Não tente "corrigir" isso — substituições devem ser raras (≤ 2 por diagrama).
- No modo escuro, use o mesmo hex, a menos que o contraste com os rótulos `paper` fique prejudicado; nesse caso, escolha um tom mais escuro para o modo escuro e documente-o como um campo `color_dark` naquele chevron.
- A substituição de cor de um chevron é independente da cor de qualquer componente pareado, mas pareá-las (mesmo hex no chevron + barra) é a forma convencional de fazer a coluna ser "lida" como uma única preocupação.

### 2.3 Zona de fontes (tracejada, externa)

```
sources_x          = 4
sources_y          = 40
sources_w          = x_boundaries[1] - 8           # width of the first chevron, minus 4px gutter each side
sources_h          = 336
```

Traço: `rgba(35,47,62,0.20)`, `stroke-width=0.8`, `stroke-dasharray=6,3`, `rx=6`. Preenchimento da zona: `rgba(35,47,62,0.02)`.

### 2.4 Fronteira do cluster (sólida)

```
cluster_x          = x_boundaries[1] + 4           # starts at end of source zone + 4px gutter
cluster_y          = 40
cluster_w          = effective_w - cluster_x       # extends to right strip / canvas edge
cluster_h          = 336
```

Traço: `rgba(35,47,62,0.18)`, `stroke-width=1.2`, `rx=8`. Preenchimento: `rgba(35,47,62,0.02)`. Ícone K8s + rótulo em `(cluster_x + 16, 352)` (ícone) e `(cluster_x + 40, 362)` (texto).

### 2.5 Barras transversais (identidade, observabilidade, …)

Zero ou mais componentes `kind: cross-cutting` são empilhados abaixo do cluster. Cada um recebe sua própria linha de 40 px com um espaçamento de 4 px.

```
crosscuts          = [c for c in components if c.kind == "cross-cutting"]   # ordered as declared
cross_x            = 4
cross_y(k)         = 388 + k * 44                  # 388, 432, 476, …
cross_w            = effective_w - 4               # spans body width, stops at the strip margin
cross_h            = 40
```

Traço: `rgba(35,47,62,0.20)`, `stroke-width=0.8`, `rx=6`. Preenchimento: `rgba(35,47,62,0.05)`. Ícone em `(16, cross_y(k) + 10)`, nome centralizado em `(effective_w / 2, cross_y(k) + 22)`, subtítulo em `(effective_w / 2, cross_y(k) + 34)`.

*Preocupações* transversais reservadas (informativo; o usuário pode dar à barra o nome que quiser):
- **Identity / Security** — Keycloak, LDAP/AD, Okta, Auth0, provedores OIDC
- **Observability** — Prometheus + Grafana, Datadog, OpenTelemetry, Loki
- **Backup / DR** — Velero, Restic, orquestradores de snapshot
- **Governance / Lineage** — OpenMetadata, DataHub, Apache Atlas
- **Secrets / config** — Vault, Sealed Secrets, External Secrets

Cada barra transversal forma par 1:1 com um chevron vertical na faixa da direita (§5).

### 2.6 Componente barra de orquestração (dentro do cluster)

```
bar_x              = cluster_x + 12
bar_y              = 52
bar_w              = cluster_w - 24
bar_h              = 44
```

Traço: `rgba(35,47,62,0.18)`, `stroke-width=0.8`, `rx=4`. Preenchimento: `rgba(35,47,62,0.05)`. Ícone da ferramenta na extrema direita (`bar_x + bar_w - 50, 58`); nome centralizado em `(bar_x + bar_w/2, 71)`; subtítulo em `(bar_x + bar_w/2, 84)`.

### 2.7 Nós de componente (dentro do cluster)

```
node_w             = 152
node_h             = 80                            # focal same height, accent border
node_cx(N)         = chevron_cx(N.chevron)         # ← non-negotiable
node_x(N)          = node_cx(N) - node_w/2
```

Se um chevron tiver K nós atribuídos, empilhe-os verticalmente:

```
first_top_y        = 120 if any bar in this column else 64
gap                = 16
row_top(k)         = first_top_y + k * (node_h + gap)   # k = 0..K-1
```

**Nó focal:** `fill="rgba(255,153,0,0.08)"`, `stroke="#EC7211"`, `stroke-width=1.2`. Texto do título na cor `accent`. Todos os outros nós: preenchimento branco, `stroke=rgba(35,47,62,0.25)`, `stroke-width=1`.

Badge de papel no canto superior esquerdo em `(node_x+8, node_y+6)`, com 12 de altura. Ícone no canto superior direito em `(node_x+node_w-32, node_y+6)`, 24×24, monocromático via `currentColor`. Nome centralizado em `(node_cx, node_y+44)`, tamanho 11, sans semibold. Subtítulo em `(node_cx, node_y+56)`, tamanho 8, mono `muted`.

### 2.8 Nós de fonte (dentro da zona tracejada)

```
src_node_w         = sources_w - 8
src_node_h         = 64                            # uniform; chosen to fit ≤ 4 sources
src_node_x         = sources_x + 4
src_first_top_y    = 60
src_gap            = 16
src_row_top(k)     = src_first_top_y + k * (src_node_h + src_gap)
```

Mesmo padrão de badge de papel / ícone / nome / subtítulo dos nós do cluster (ícone em `src_node_x+54, row_top(k)+6`; nome centralizado em `src_node_x+src_node_w/2, row_top(k)+42`; subtítulo uma linha abaixo). Texto do badge de papel: `EXT`. Limite de 4 fontes; para mais, divida em um diagrama separado.

### 2.9 Faixa da direita — chevrons verticais

```
strip_x            = 1000 - right_strip_w       # 972 when present
strip_w            = 28
verticals          = [c for c in chevrons if c.vertical or c.name in reserved_names]
strip_y_top        = 40
strip_y_bot        = max(428, 388 + n_cross * 44 - 4)   # extends to last crosscut row (see §2.1)
strip_h_total      = strip_y_bot - strip_y_top
heights            = [floor_to_4(strip_h_total / len(verticals))] * len(verticals)
heights[-1]       += strip_h_total - sum(heights)       # last absorbs remainder
```

Exemplos:
- 2 verticais (Orchestration + Security), 1 transversal → `heights = [192, 196]`, layout `[40..232, 232..428]`.
- 3 verticais (Orchestration + Security + Observability), 2 transversais → `strip_y_bot = 472`, `strip_h_total = 432`, `heights = [144, 144, 144]`, layout `[40..184, 184..328, 328..472]`.

Bordas adjacentes compartilham o mesmo y (sem espaçamento), assim como os chevrons horizontais compartilham o x em sua fronteira.

**Formas dos polígonos** (fluxo de cima para baixo, espelha a horizontal de §2.2):
- Primeiro (mais acima): topo plano, ponta embaixo — `(strip_x, y0) (strip_x+strip_w, y0) (strip_x+strip_w, y1-12) (strip_x+strip_w/2, y1) (strip_x, y1-12)`
- Do meio: entalhe em cima, ponta embaixo — `(strip_x, y0) (strip_x+strip_w/2, y0+12) (strip_x+strip_w, y0) (strip_x+strip_w, y1-12) (strip_x+strip_w/2, y1) (strip_x, y1-12)`
- Último (mais abaixo): entalhe em cima, base plana — `(strip_x, y0) (strip_x+strip_w/2, y0+12) (strip_x+strip_w, y0) (strip_x+strip_w, y1) (strip_x, y1)`

Os preenchimentos alternam `#232F3E` / `#3d4460` (mesma paleta dos horizontais). Rótulos: mono na cor `paper`, `font-size=7`, `letter-spacing=0.14em`, **rotacionados −90°**, ancorados em `(strip_x + strip_w/2, (y0+y1)/2)`.

Os chevrons verticais respeitam a substituição de `color` por chevron documentada em §2.2 — aplique o hex ao preenchimento do polígono e mantenha o rótulo rotacionado na cor `paper`. Combine a substituição com o mesmo hex na barra/transversal pareada do chevron para uni-los visualmente como uma única preocupação.

---

## 3. Regras de conectores (obrigatórias)

Estas regras são inegociáveis. Escolha o estilo **automaticamente** a partir da topologia — não permita que o usuário substitua o estilo em arestas que tocam o nó focal ou que se originam de uma barra.

| `style` | Traço | Largura | Tracejado | Marcador | Quando é obrigatório |
|---|---|---|---|---|---|
| `primary` | `#EC7211` | 1.2 | — | `arrow-accent` | Toda aresta cuja extremidade é o nó `focal`. |
| `secondary` | `#545B64` | 1.0 | — | `arrow` | Padrão para fonte→componente e componente→componente quando nenhuma extremidade é focal. |
| `trigger` | `#545B64` | 1.0 | `4,3` | `arrow-sm` | Toda aresta que se origina de um componente `kind: bar`. |
| `query` | `rgba(35,47,62,0.30)` | 1.0 | `4,3` | `arrow` | Arestas de leitura de retorno (ex.: focal ↔ Trino). |

**Bloco defs** (obrigatório, exatamente estes quatro marcadores):

```svg
<defs>
  <marker id="arrow"        markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#545B64"/></marker>
  <marker id="arrow-accent" markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#EC7211"/></marker>
  <marker id="arrow-sm"     markerWidth="6" markerHeight="5" refX="5" refY="2.5" orient="auto"><polygon points="0 0, 6 2.5, 0 5" fill="#545B64"/></marker>
  <marker id="arrow-dim"    markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="rgba(35,47,62,0.45)"/></marker>
</defs>
```

### 3.1 Lados de saída / entrada (inegociável)

| Tipo de aresta | Lado de saída da origem | Lado de entrada do destino |
|---|---|---|
| Fonte → nó do cluster | **direita** do nó de fonte | **esquerda** do destino |
| Componente → componente (dentro do cluster) | **direita** | **esquerda** |
| Componente barra → nó | **base** da barra | **topo** do nó |
| Barra transversal | — (não emite arestas) | — |
| Chevron vertical | — (apenas rótulos; não emite arestas) | — |

### 3.2 Roteamento

- Cotovelos ortogonais, **no máximo duas dobras** por caminho.
- Use um canto com Q-bezier de raio 8 px em cada dobra.
- Ordem Z: desenhe **todos os conectores antes de qualquer retângulo de nó** (para que o preenchimento dos nós mascare as pontas das linhas).
- Exatamente um `marker-end` por `<path>` / `<line>`. Nunca `marker-start` e `marker-end` juntos.
- Rótulos: todo conector `primary` e `secondary` recebe um rótulo (mono pequeno, com uma máscara retangular opaca preenchida com `paper` atrás). Conectores `trigger` e `query` não têm rótulo.

### 3.3 Cruzamentos

- Evite. Redirecione pelo tronco do divisor de chevrons (§4) antes de aceitar um cruzamento.
- Se for inevitável, o caminho desenhado em segundo lugar recebe um salto em arco (hop) de 6 px sobre o primeiro.

### 3.4 Substituição de cor de componente

Um componente pode declarar um `color: "#hex"` opcional (string de cor CSS). A substituição recolore **apenas** o contêiner e o conteúdo do componente — os conectores nunca são recoloridos. As arestas mantêm o estilo definido pela topologia em §3.

**Onde a cor é aplicada** (dado `C = color`):

| Elemento | Modo claro | Modo escuro |
|---|---|---|
| Preenchimento do contêiner (corpo `rect`) | `rgba(C, 0.06)` | `rgba(C, 0.10)` |
| Traço do contêiner | `rgba(C, 0.35)` (`stroke-width=1` para nós, `0.8` para barras) | `rgba(C, 0.45)` |
| Traço do badge de papel (nós) | `rgba(C, 0.40)` | `rgba(C, 0.55)` |
| Texto do badge de papel (nós) | `rgba(C, 0.85)` | `rgba(C, 1.0)` |
| Preenchimento / traço do ícone | `C` | clareie `C` em ~15% (ou use `C` se já for claro) |
| Texto do nome | `C` | `C` clareado |
| Texto do subtítulo | **inalterado** (`muted`) | **inalterado** (`muted`) |
| Conectores que tocam este componente | **inalterados** (ainda definidos pela topologia) | **inalterados** |

O subtítulo permanece `muted` porque é um metadado parentético — apenas a identidade principal (nome + ícone + borda) carrega o sinal de cor.

**Regras**:

- **Nunca no nó focal.** O nó focal já é colorido com o `accent` (§2.7). Um `color` no nó focal é ignorado — o `accent` prevalece.
- **Nunca em nós de fonte.** As fontes ficam fora do cluster e permanecem neutras.
- **Limite de 2 componentes com cor personalizada** por diagrama (além do focal). Três ou mais elementos coloridos apagam o sinal — o mesmo motivo pelo qual §1 limita o `accent` a 1–2 elementos.
- **Sem cor em conectores.** Se você sentir vontade de colorir uma aresta, o certo é escolher outro `style` de §3, e não substituir a cor.

**Usos semânticos** (recomendados):
- `#b85450` (vermelho-ferrugem) — Security / Identity (Keycloak, Vault)
- `#5a7d9a` (azul-ardósia) — Observability (Prometheus, Datadog)
- `#7a8c47` (verde-oliva) — Governance / Lineage (OpenMetadata)
- `#8c6d3f` (marrom-quente) — Backup / DR

Atenha-se a estas, a menos que a marca do usuário exija outra coisa. Um hex aleatório por componente é exatamente o modo de falha que esta skill evita.

---

## 4. Regras de ramificação de blocos (fan-out)

O maior risco individual para a reprodutibilidade. Fixe estas regras e o diagrama se torna previsível.

### 4.1 Fan-out de fonte (uma fonte → N componentes)

```
exit_x   = source.right
trunk_x  = cluster_x - 8                     # 4-px gutter before cluster border
```

Caminho por destino: `M exit_x,source_cy → H trunk_x → V target_cy → H target.left`. Use cantos com Q-bezier.

### 4.2 Fan-out de componente (um componente → N componentes)

```
exit_x   = node.right
trunk_x  = x_boundaries[index(source.chevron) + 1] + 4   # 4 px past the chevron divider
```

Caminho por destino: `M exit_x,source_cy → H trunk_x → V target_cy → H target.left`.

### 4.3 Limite de fan-out

**No máximo 3 arestas de saída por nó.** Acima de 3, introduza um hub (normalmente o nó `focal`). A faixa de chevrons é a legenda; se um nó está distribuindo (fan-out) para quatro destinos a jusante, ele é secretamente o hub — torne isso explícito.

### 4.4 Descidas da barra (Airflow → N nós)

```
drop_x(target) = target.cx
drop_y_start   = bar.bottom
drop_y_end     = target.top
```

Linha vertical reta, `style: trigger`. Uma por destino. Sem dobras — descidas da barra nunca fazem cotovelo.

### 4.5 Escalonamento vertical das fontes

Quando várias fontes se conectam ao mesmo destino único (ex.: quatro fontes → NiFi), escalone o y de entrada delas no destino:

```
entry_y(k) = target.top + 8 + k * (target_h - 16) / (N - 1)   # k = 0..N-1, evenly spaced
```

Isso evita pontas de seta sobrepostas na borda esquerda do destino.

---

## 5. Chevrons verticais — semântica

Os nomes reservados `Orchestration`, `Security`, `Observability`, `Governance`, `Backup` sempre são renderizados na faixa da direita (§2.9). Qualquer chevron com `vertical: true` é tratado como um vertical no estilo reservado, independentemente do nome. As regras:

- **Regra de pareamento (obrigatória, 1:1):** todo chevron vertical forma par com exatamente um componente de largura total, e todo componente de largura total forma par com exatamente um chevron vertical. Os dois tipos de componente que formam par:
  - `kind: bar` — fica dentro do cluster (linha superior). Convencionalmente pareado com `Orchestration`.
  - `kind: cross-cutting` — fica abaixo do cluster, uma linha por componente. Pareado com `Security`, `Observability`, `Governance` etc.

  Se as entradas declararem um chevron vertical sem um componente pareado (ou vice-versa), pare e pergunte ao usuário — o diagrama está incompleto.

- **Restrição de contagem:** `len(verticals) == len(bars) + len(crosscuts)`. A faixa da direita é dividida igualmente entre todos os verticais (§2.9), então o alinhamento visual entre um chevron e sua barra/linha é aproximado — o que importa é o *rótulo*, não a correspondência exata de pixels em y.

- **Convenção de ordenação:** declare os verticais de cima para baixo nesta ordem: primeiro os pareados com barra (Orchestration), depois os pareados com transversais, na mesma ordem em que as transversais aparecem abaixo do cluster. Isso mantém a ordem de leitura visual consistente.

- **Sem arestas:** chevrons verticais não emitem conectores. Eles são *rótulos para uma coluna de preocupação transversal*.

- **Sem posicionamento de nós:** nenhum `kind: node` pode ser atribuído a um chevron vertical. Os nós sempre pertencem a uma fase horizontal.

- **Presença da faixa da direita:** se existir algum chevron vertical, a faixa da direita fica reservada (`effective_w = 964`) e **todas** as larguras dos chevrons horizontais e a geometria do cluster encolhem de acordo. Não desenhe um chevron vertical por cima do cluster.

O contrato visual: a coluna do chevron vertical "possui" visualmente a barra/linha transversal na sua faixa aproximada de y. Orchestration (topo da faixa) ↔ barra Airflow (topo do cluster). Security ↔ barra Identity. Observability ↔ barra de monitoramento (abaixo da identidade). E assim por diante.

---

## 6. Modo escuro

Quando `dark: true`, troque estes tokens:

| Token | Claro | Escuro |
|---|---|---|
| Papel da página (paper) | `#ffffff` | `#1c1f2e` |
| Tinta (ink) | `#232F3E` | `#ffffff` |
| Texto atenuado (muted) | `#545B64` | `rgba(255,255,255,0.65)` |
| Preenchimento escuro do chevron | `#232F3E` | `#3d4460` |
| Preenchimento claro do chevron | `#3d4460` | `#4a5270` |
| Rótulo do chevron | `#ffffff` | `#ffffff` (inalterado) |
| Borda tracejada | `rgba(35,47,62,0.20)` | `rgba(255,255,255,0.22)` |
| Borda do cluster | `rgba(35,47,62,0.18)` | `rgba(255,255,255,0.18)` |
| Preenchimento do nó | branco | `rgba(255,255,255,0.06)` |
| Traço do nó | `rgba(35,47,62,0.25)` | `rgba(255,255,255,0.20)` |
| Preenchimento focal | `rgba(255,153,0,0.08)` | `rgba(255,153,0,0.12)` |
| Traço focal | `#EC7211` | `#FF9900` |
| Conector de destaque (accent) | `#EC7211` | `#FF9900` |
| Padrão de pontos | `rgba(35,47,62,0.10)` | `rgba(255,255,255,0.10)` |

---

## 7. Checklist de reprodutibilidade (o *taste gate*)

Antes de emitir o SVG, verifique **cada** item. Se algum falhar, corrija — não entregue.

1. Todo `node.cx` do cluster é igual ao `cx` do seu chevron (§2.2 + §2.7). É isso que faz da faixa de chevrons uma legenda de verdade.
2. Toda `width` de chevron é múltiplo de 4 e ≥ 120.
3. A faixa reservada à direita (28 px) existe **se e somente se** algum chevron vertical for declarado. Se sim, `effective_w = 972`; se não, `effective_w = 1000`.
4. Exatamente **um** nó `focal`. Se `focal` não estiver definido nas entradas, use por padrão o primeiro `kind: node` sob o chevron "Storage".
5. Toda aresta cuja extremidade é o nó focal usa `style: primary` (traço `accent` + marcador `arrow-accent`).
6. Toda aresta que se origina de um componente `kind: bar` usa `style: trigger` (tracejado + `arrow-sm`).
7. A barra transversal (se houver) **não** emite arestas.
8. Nenhum nó tem > 3 arestas de saída, ou, se tiver, é o `focal` / hub declarado.
9. Todos os conectores `<path>` e `<line>` são emitidos **antes** de qualquer `<rect>` de nó (ordem Z).
10. Cada chevron vertical forma par **1:1** com exatamente um componente `bar` ou `cross-cutting` (regra de pareamento de §5). `len(verticals) == len(bars) + len(crosscuts)`.
11. `viewBox_h = max(540, strip_y_bot + 112)` — aumente o canvas quando várias transversais forem declaradas para que a legenda continue cabendo.
12. Cores personalizadas de componentes (§3.4) se aplicam apenas a contêiner + ícone + nome; os conectores continuam definidos pela topologia. Limite de 2 componentes com cor personalizada além do focal.
11. O diagrama passa em SKILL.md §9 (grade de 4 px; ≤ 2 elementos `accent`; mono apenas para conteúdo técnico; linhas finas (hairlines); sem sombras; sem `rounded-2xl`).

---

## 8. Antipadrões

- Faixa de chevrons omitida — ela é a chave que mapeia colunas visuais para fases funcionais.
- Centro x do nó fora do chevron (§7 #1) — quebra o contrato de "faixa como legenda".
- Chevron vertical desenhado sobre o cluster (sobreposição) em vez de em uma faixa reservada à direita.
- Mais de um nó focal — MinIO/S3 (ou qualquer que seja o hub de armazenamento) é *o* ponto focal.
- Zona externa com borda sólida — a borda tracejada é o sinal de que esses componentes estão fora do cluster.
- Barra de identidade dentro da fronteira do cluster — ela se aplica a todos os componentes e deve ocupar toda a largura do canvas.
- Chevron vertical sem um componente barra/transversal pareado — veja a regra de pareamento de §5.
- Arestas de componente barra desenhadas sólidas — gatilhos de orquestração devem ser tracejados.
- Fonte distribuindo (fan-out) para >3 componentes sem um hub.

---

## 9. Exemplos

- `assets/example-high-level.html` — apenas horizontal, 5 fases, skin claro.
- `assets/example-high-level-dark.html` — o mesmo, skin escuro.
- `assets/example-high-level-full.html` — o mesmo, com moldura de cartão editorial.
- `assets/example-high-level-vertical.html` — adiciona chevrons verticais Orchestration + Security, barra Airflow e transversal Keycloak. **Renderização de referência do padrão paramétrico completo.**
- `assets/example-high-level-vertical-dark.html` — padrão vertical, skin escuro.
- `assets/example-high-level-vertical-full.html` — padrão vertical, com moldura de cartão editorial.
