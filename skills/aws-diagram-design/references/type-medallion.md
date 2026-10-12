# Medallion

**Ideal para:** documentar um layout de armazenamento de dados em várias camadas, em que cada camada é um *nível de qualidade / acesso* distinto do mesmo conjunto de dados — normalmente zona de aterrissagem bruta, anonimizada, staging/limpa, indicadores de negócio agregados e arquivo frio. Usado quando o leitor precisa ver de relance *o que cada bucket contém*, *quem escreve nele*, *com qual ferramenta e formato* e *como os dados são promovidos entre as camadas*.

Prefira **Process** se o assunto for um fluxo de trabalho com raias por papel. Prefira **High-Level** se o assunto for a arquitetura do cluster, e não a organização das camadas de armazenamento.

Este tipo é **paramétrico** — o esquema de entradas em §1 determina cada coordenada por meio das fórmulas em §2. Duas gerações a partir das mesmas entradas devem produzir SVGs visualmente idênticos. As formas das regras espelham `type-process.md` e `type-data-flow.md`, de modo que a substituição de cor, a regra focal e o checklist de reprodutibilidade se leem de forma idêntica entre os tipos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
title:    "Five-Tier Medallion Architecture"
subtitle: "Quarterly survey through Raw → Anonymized → Staging → Aggregated → Archive"

tiers:                                # 3..6 tier columns, ordered left → right
  - { name: "Raw",        bucket: "raw-bucket",        style: "outer",
      fields: { tool: "NiFi · raw write",          format: "CSV · Parquet · JSON", writer: "Data Engineer",
                example: ["Q1 dump · w/ PII", "verbatim CAPI export"] } }
  - { name: "Anonymized", bucket: "anon-bucket",       style: "default",
      fields: { tool: "Trino INSERT",              format: "Iceberg · partitioned", writer: "Data Engineer",
                example: ["no name · address", "stable household ID"] } }
  - { name: "Staging",    bucket: "staging-bucket",    style: "default",  color: "#c9a23a",   # warm yellow — analytical working zone
      fields: { tool: "Trino · JupyterHub",        format: "Iceberg · cleaned",     writer: "Data Scientist",
                example: ["weighted records", "harmonised codings"] } }
  - { name: "Aggregated", bucket: "aggregated-bucket", style: "focal",  focal: true,
      fields: { tool: "Trino INSERT · SAS JDBC",   format: "Iceberg · indicators",  writer: "Data Scientist",
                example: ["unemployment rate", "labour participation"] } }
  - { name: "Archive",    bucket: "archive-bucket",    style: "cold",
      fields: { tool: "MinIO lifecycle",           format: "cold tier · immutable", writer: "Data Administrator",
                example: ["historical Q1–Q4 sets", "5+ years retained"] } }

example_label: "Quarterly survey example" # bottom field label (varies per domain)

promotions:                           # adjacent-tier arrows; len = n_tiers - 1
  - { from: 0, to: 1, label: "PII REMOVE",    style: "normal"    }
  - { from: 1, to: 2, label: "CLEAN+WEIGHT",  style: "normal"    }
  - { from: 2, to: 3, label: "AGGREGATE",     style: "focal"     }    # auto-accent because target is focal
  - { from: 3, to: 4, label: "LIFECYCLE",     style: "lifecycle" }    # dashed

paths:                                # 0..2 write-method cards at the bottom (optional)
  - { tag: "SQL PATH",      title: "Trino INSERT INTO … SELECT",
      sub: "filter · reshape · join · aggregate — set-based transforms" }
  - { tag: "NOTEBOOK PATH", title: "DuckDB + Python/R in JupyterHub",
      sub: "stats · ML · interactive analysis — row-iterative work" }

dark: false
```

**Semântica dos campos reservados:**
- `tiers[i].style` — um de `outer`, `default`, `focal`, `cold`. Determina a paleta de preenchimento/traço do cartão (§2.4).
- `tiers[i].focal: true` — exatamente **uma** camada pode declarar isto. Substitui `style` por `focal` e muda automaticamente para `focal` a seta de promoção que *entra* nesta camada.
- `tiers[i].fields` — `{tool, format, writer, example}`. `example` é uma lista de 1 ou 2 itens; o título da seção usa `example_label`.
- `tiers[i].color` — substituição de cor `"#hex"` opcional por camada. Veja §4.
- `promotions[].style` — `normal` | `focal` | `lifecycle`. A regra de conectores (§3) vincula cada estilo a traço / tracejado / marcador fixos.
- `paths` — 0–2 entradas. Com 0 entradas, a linha inferior é omitida e `viewBox_h` encolhe de acordo.

---

## 2. Fórmulas de layout — geometria determinística

```
# Tier dimensions
tier_w           = 172
tier_h           = 380
tier_gap         = 16
left_pad         = 16
right_pad        = 100
n_tiers          = len(tiers)

# Canvas
viewBox_w        = left_pad + n_tiers * tier_w + (n_tiers - 1) * tier_gap + right_pad
                                                    # 5 tiers → 16 + 860 + 64 + 100 = 1040
arc_band_h       = 80                               # space above tiers reserved for promotion arcs
path_h           = 56
path_gap         = 16                               # gap between tier row and path row
bottom_pad       = 16
viewBox_h        = arc_band_h + tier_h + (path_gap + path_h if paths else 0) + bottom_pad
                                                    # with paths → 80+380+72+16 = 548
                                                    # without paths → 80+380+16 = 476

# Tier positions
tier_x(i)        = left_pad + i * (tier_w + tier_gap)    # 16, 204, 392, 580, 768
tier_y           = arc_band_h                            # 80 — tier tops sit just below the arc band
tier_cx(i)       = tier_x(i) + tier_w/2                  # 102, 290, 478, 666, 854

# Promotion arcs (between adjacent tiers — over the top, anchored at tier top-centers)
arc_src_x(i)     = tier_cx(i)                            # top-center of tier i      (102, 290, 478, 666)
arc_dst_x(i)     = tier_cx(i+1)                          # top-center of tier i+1    (290, 478, 666, 854)
arc_peak_x(i)    = (arc_src_x(i) + arc_dst_x(i)) / 2     # midpoint                  (196, 384, 572, 760)
arc_label_y      = 50                                    # label sits inside the arc, 30px below tier top

# Path row (bottom)
path_y           = tier_y + tier_h + path_gap            # 476
path_w           = (viewBox_w - 2*left_pad - path_gap) / 2 if len(paths) == 2 else (viewBox_w - 2*left_pad)
                                                          # Canonical 5-tier shape uses path_w=460 explicitly (see §2.5)
```

### 2.1 Fundo

Preenchimento sólido `paper` em todo o viewBox. Sem padrão de pontos.

### 2.2 Cartão de camada (172 × 380)

Cada camada é renderizada como um cartão retangular de cantos arredondados, com uma faixa de cabeçalho tingida, um nome de bucket centralizado, quatro linhas de campos rotulados e uma seção `example_label` separada perto da base.

```
tier_x(i),  tier_y       =  card top-left  (tier_y = 80, just below the arc band)
header_band_h            = 40            # band from y=tier_y to y=tier_y+40 (i.e., 80..120)
header_band_extra        = 10            # 10-px extension below band, same tint

# Inside the card (absolute y; tier_y = 80):
title_text            at (tier_cx(i), 106)                # node-name role, 13px, weight 700, ink
bucket_text           at (tier_cx(i), 144)                # sublabel role, muted (accent on focal tier)

field_x               = tier_x(i) + 16                    # 16-px left inset for field text
field_w               = 140                               # 172-px tier_w minus two 16-px insets
field rows (absolute y):
  tool_label  at 180,    tool_value  at 186 (foreignObject, height 24)
  format_label at 220,   format_value at 226 (foreignObject, height 24)
  writer_label at 260,   writer_value at 266 (foreignObject, height 24)
  # gap (open whitespace below writer row, above the example section)
  example_label_text at 360,  example_line_0 at 374,  example_line_1 at 388
```

**Regra de quebra de linha dos valores de campo:** os valores de campo (tool / format / writer) são renderizados dentro de um `<foreignObject>` SVG com um `<div>` HTML, para que quebrem a linha automaticamente quando o texto exceder 140 px. Cada `foreignObject` tem 140 de largura × 24 de altura (cabem 2 linhas no papel `sublabel` com line-height 1.25). O espaço de 26 px até o rótulo do campo seguinte absorve a segunda linha sem problemas.

```svg
<foreignObject x="{field_x}" y="{value_top}" width="140" height="24">
  <div xmlns="http://www.w3.org/1999/xhtml"
       style="font-family: {sublabel}; color: {muted}; line-height: 1.25;">
    {field_value}
  </div>
</foreignObject>
```

A declaração de namespace HTML no `<div>` é necessária para que o SVG renderize o conteúdo inline. Navegadores e Playwright/Chromium renderizam isso fielmente; se o seu destino de exportação não suportar `<foreignObject>` (algumas versões antigas do Inkscape), divida manualmente os valores longos em duas linhas `<tspan>`.

Os rótulos de campo usam o papel `node-name` a 11px em `ink`. Os valores de campo usam o papel `sublabel` em `muted`. O bucket e os valores de campo podem ser recoloridos pela substituição de `color` (§4).

### 2.3 Estilos de camada

Quatro estilos canônicos, escolhidos por camada via `tiers[i].style`. Mapeamento padrão se `style` for omitido: camada 0 → `outer`, última camada → `cold`, camada focal (se houver) → `focal`, demais → `default`.

| `style` | Preenchimento do cartão | Traço do cartão | Preenchimento da faixa de cabeçalho | Texto do bucket | Texto dos valores de exemplo |
|---|---|---|---|---|---|
| `outer` | `#FFFFFF` | `muted` 1.0 sólido | `muted @ 0.10` | `muted` | `muted` |
| `default` | `#FFFFFF` | `ink` 1.0 sólido | `ink @ 0.06` | `muted` | `muted` |
| `focal` | `accent @ 0.07` | `accent` 1.6 sólido | `accent @ 0.14` | `accent` | `accent` |
| `cold` | `paper-2` | `muted` 1.0 tracejado `5,3` | `muted @ 0.18` | `muted` | `muted` |

`rx = 6` em todos os retângulos de cartão.

**Nota sobre o estilo focal:** o tratamento `accent` da camada focal se propaga — o texto do bucket e as linhas de valores de exemplo são renderizados em `accent`. Os outros valores de campo (tool/format/writer) permanecem `muted`; apenas o nome do bucket e a carga de exemplo carregam o sinal focal, para que o cartão da camada não se afogue totalmente em coral.

### 2.4 Arcos de promoção (por cima das camadas)

Cada promoção é um **arco Bézier cúbico** ancorado no **centro do topo** de cada camada adjacente — de `(tier_cx(i), tier_y)` a `(tier_cx(i+1), tier_y)`. O arco sobe para a `arc_band` de 80 px acima dos cartões, atingindo o pico em y ≈ 20. Tanto o conector quanto seu rótulo permanecem totalmente visíveis — sem máscaras `paper`, sem sobreposição com o conteúdo dos cartões.

```svg
<path d="M {tier_cx(i)},{tier_y} C {tier_cx(i)},0 {tier_cx(i+1)},0 {tier_cx(i+1)},{tier_y}"
      fill="none" stroke="…" stroke-width="…" marker-end="…"/>
```

Concreto para a forma canônica de 5 camadas (`tier_y = 80`, centros das camadas em x = 102, 290, 478, 666, 854):
- 0→1: `M 102,80 C 102,0 290,0 290,80`
- 1→2: `M 290,80 C 290,0 478,0 478,80`
- 2→3: `M 478,80 C 478,0 666,0 666,80`  (focal — `accent`)
- 3→4: `M 666,80 C 666,0 854,0 854,80`  (lifecycle — tracejado)

A geometria cúbica: âncora em y = 80 (topo da camada), ponto de controle em y = 0 (topo do viewBox). O pico da curva em t=0.5 fica em y ≈ 20 (calculado por `0.125·80 + 0.375·0 + 0.375·0 + 0.125·80 = 20`). Cada arco abrange um passo completo de camada (188 px no layout canônico), dando ao conector uma excursão vertical claramente visível.

**Orientação do marcador:** `marker-end` com `orient="auto"` rotaciona a seta para acompanhar a tangente do caminho no ponto final. O ponto de controle fica diretamente acima da âncora, então a tangente na chegada aponta reto **para baixo** — a ponta da seta entra de forma limpa no centro do topo da camada *i+1*, apontando para a faixa de cabeçalho.

**Âncoras encadeadas:** arcos consecutivos compartilham seus pontos de encontro (o arco 0→1 termina no mesmo `(tier_cx(1), 80)` onde o arco 1→2 começa). Visualmente, o centro do topo de cada camada funciona como uma "junta" — os dados chegam ao topo do cartão, são transformados dentro dele e saem pelo topo em direção à próxima camada. O mergulho da ponta da seta somado à emergência reta para cima do arco seguinte é lido como um único movimento de entrega da carga.

| `style` | Traço | Largura | Tracejado | Marcador |
|---|---|---|---|---|
| `normal` | `muted` | 1.4 | — | `arrow` |
| `focal` | `accent` | 1.6 | — | `arrow-accent` |
| `lifecycle` | `muted` | 1.4 | `4,3` | `arrow` |

**Regras de estilo automático:**
- Se `promotions[k].to` referenciar a **camada focal**, o estilo é promovido automaticamente a `focal` (`accent`, largura 1.6, marcador `arrow-accent`).
- Se `promotions[k].to` referenciar uma camada com **substituição de `color`** (§4), a seta herda esse hex — traço = `C`, preenchimento do rótulo = `C`, e o `marker-end` usa um marcador de cor correspondente (ex.: `arrow-yellow` para `#c9a23a`). A largura permanece 1.4 — a substituição de cor é um sinal de "preocupação", não uma promoção focal. Setas lifecycle/tracejadas mantêm o tracejado, mas adotam a cor.
- O focal prevalece se ambas se aplicarem (uma camada colorida marcada como focal ainda usa `accent`).

**Rótulo dentro do arco:**
- Ancorado em `(arc_peak_x(k), arc_label_y)` = `((arc_src_x + arc_dst_x) / 2, 50)`.
- Papel `arrow-label` a 10px com `letter-spacing=0.08em`, em maiúsculas. A cor acompanha o traço da seta.
- **Nenhum retângulo de máscara é necessário** — a curva cúbica atinge o pico em y ≈ 20 e o rótulo fica em y=50, bem abaixo da curva. O rótulo flutua dentro do espaço aberto *envolvido* pelo arco, lendo-se como "X se transforma em Y", com o próprio arco servindo de moldura visual.

Para espaços menores entre camadas (se `tier_gap` for substituído por um valor abaixo do padrão de 16 px), o `arc_inset` das âncoras do arco pode precisar diminuir proporcionalmente para manter o arco visível.

### 2.5 Linha de caminhos (inferior, opcional)

Até **2** cartões de método de escrita. A forma canônica de 5 camadas (com `arc_band_h = 80`):

```
path_y      = 476       # tier_y + tier_h + path_gap = 80 + 380 + 16
path_h      = 56
path_x[0]   = 16
path_w[0]   = 460
path_x[1]   = 16 + 460 + 16 = 492
path_w[1]   = 460
```

(Ambos os caminhos ficam com 460 de largura, apesar de o viewBox ter 1040 — o preenchimento direito é tomado da faixa de cartões de camada, não da faixa de caminhos. Mantenha `path_w=460` para a forma canônica de 5 camadas. Para outras quantidades de camadas, derive `path_w = (viewBox_w - 2*left_pad - path_gap) / 2`.)

Conteúdo por cartão:
- Retângulo contêiner: preenchimento branco, traço `ink @ 0.20` com largura 1, `rx=6`.
- Chip de tag: retângulo em `(path_x + 8, path_y + 6)`, `h=12 rx=2`, preenchimento transparente, traço `ink @ 0.30` com largura 0.8. Texto da tag centralizado dentro, no papel `eyebrow`, com letter-spacing 0.08em, em `ink`.
- Título em `(path_x + 80, path_y + 30)`: papel `node-name` a 11px, `ink`.
- Sub em `(path_x + 80, path_y + 46)`: papel `sublabel`, `muted`.

---

## 3. Regras de conectores (obrigatórias)

Três estilos, vinculados à topologia. Espelhe §3 de `type-process.md` para que a regra se leia de forma idêntica.

```svg
<defs>
  <marker id="arrow"        markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="{muted}"/></marker>
  <marker id="arrow-accent" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="{accent}"/></marker>
  <!-- Per-color markers: declare one per custom tier color in use. -->
  <marker id="arrow-yellow" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#c9a23a"/></marker>
</defs>
```

Adicione um novo `<marker>` para cada substituição de `color` usada no diagrama. Convenção de nomes: `arrow-{semantic}` (ex.: `arrow-yellow`, `arrow-slate`, `arrow-red`) — corresponde à paleta recomendada, para que o id do marcador seja lido com clareza no código-fonte.

**Ordem Z:** os arcos de promoção são desenhados **antes** de qualquer retângulo de cartão de camada, para que os cartões fiquem por cima e qualquer excesso do arco seja mascarado dentro dos cartões.

**Regra de forma do arco:** as promoções do medallion são sempre **arcos cúbicos por cima** da faixa de camadas, ancorados no **centro do topo** das camadas de origem e destino; pontos de controle diretamente acima das âncoras em y=0. Nada de linhas horizontais "pelo vão" — o arco por cima é o que torna o conector e o rótulo claramente visíveis.

---

## 4. Substituição de cor de componente

Qualquer entrada de camada ou de caminho aceita um `color: "#hex"` opcional. Espelha `type-process.md` §4 / `type-data-flow.md` §4.

### 4.1 `color` por camada

Aplicado a:

| Elemento | Claro | Escuro |
|---|---|---|
| Preenchimento do cartão | `rgba(C, 0.07)` | `rgba(C_light, 0.10)` |
| Traço do cartão | `C` (largura 1.4) | `C_light` (largura 1.4) |
| Preenchimento da faixa de cabeçalho | `rgba(C, 0.14)` | `rgba(C_light, 0.18)` |
| Texto do título | `ink` (inalterado — o título continua legível) | `ink` (inalterado) |
| Texto do bucket | `C` | `C_light` |
| Valores de exemplo | `C` | `C_light` |
| Rótulos / valores de campo | **inalterados** (`ink` / `muted`) | **inalterados** |
| Conectores que tocam esta camada | **inalterados** — definidos pela topologia | **inalterados** |

`C_light` = o mesmo hex clareado em ~15% para contraste no modo escuro.

### 4.2 `color` por caminho

Substitui o traço do cartão de caminho por `rgba(C, 0.45)` e o traço do chip de tag por `rgba(C, 0.55)`. O texto da tag e o texto do título usam `C`. O sub permanece `muted`.

### 4.3 Regras

- **Nunca em camadas focais.** O `accent` já carrega esse sinal — um `color` na camada focal é ignorado.
- **Nunca na camada `cold` em adição ao seu tratamento tracejado.** Escolha o cold tracejado ou uma cor personalizada, não ambos.
- **Limite de 2 elementos com cor personalizada** por diagrama (camada ou caminho), além da camada focal.
- **Setas de promoção herdam a cor da camada de destino** (regra de estilo automático de §3). Um `color: "#c9a23a"` na camada Staging significa que o arco CLEAN+WEIGHT que chega em Staging também é renderizado em amarelo — conector, rótulo e ponta de seta combinam. Isso mantém a coerência visual: a camada colorida e o fluxo que entra nela são lidos como um único grupo de "preocupação". As setas **não** herdam a cor da camada de origem — apenas da de destino —, então o arco que *sai* de uma camada colorida volta a `muted` (ou à cor/estilo do próximo destino).

### 4.4 Paleta semântica (recomendada)

A mesma paleta dos outros tipos paramétricos, para que um leitor que percorra vários diagramas veja as mesmas cores significando as mesmas coisas:

- `#b85450` vermelho-ferrugem — Security / Identity / Governance (camadas com PII, camadas de auditoria)
- `#5a7d9a` azul-ardósia — Observability / Quality (camadas validadas, zonas monitoradas)
- `#7a8c47` verde-oliva — Data Products / Publication (agregados voltados ao consumidor, camadas de publicação pública)
- `#c9a23a` amarelo-quente / dourado — zonas analíticas / de trabalho (camada de staging, sandbox de cientistas, superfície de cálculo intermediário)
- `#8c6d3f` marrom-quente — Backup / DR / Archive (estilo alternativo para camada fria)

---

## 5. Regra focal

Exatamente **uma** camada focal por diagrama. Por padrão, é a camada marcada com `focal: true` nas entradas; se nenhuma estiver marcada, usa-se a camada pivô analítica (normalmente `Aggregated` ou qualquer camada que os consumidores a jusante consultem).

A camada focal:
- Usa `style: focal` (preenchimento `accent` + traço 1.6 + faixa de cabeçalho `accent`).
- Renderiza o texto do bucket e as linhas de valores de exemplo em `accent`.
- Tem sua seta de promoção de **entrada** promovida automaticamente a `style: focal` (`accent`).
- Tem sua seta de promoção de **saída** (se houver) — normalmente para o arquivo frio — mantida no estilo declarado pelo usuário (geralmente `lifecycle` tracejado).

Se zero ou >1 camadas tiverem `focal: true`, pare e pergunte ao usuário.

---

## 6. Modo escuro

| Token | Claro | Escuro |
|---|---|---|
| Paper | `paper` | `ink` |
| Ink | `ink` | `paper` |
| Muted | `muted` | `soft` |
| Accent | `accent` | `accent` |
| Fog (preenchimento da camada fria) | `paper-2` | `paper @ 0.06` |
| Branco (preenchimento padrão do cartão) | `#FFFFFF` | `paper @ 0.04` |
| Traço `ink` do cartão (estilo padrão) | `ink` | `paper @ 0.30` |
| Tinta `ink` da faixa de cabeçalho | `ink @ 0.06` | `paper @ 0.08` |
| Tinta `muted` da faixa de cabeçalho | `muted @ 0.10` | `soft @ 0.16` |
| Tinta cold da faixa de cabeçalho | `muted @ 0.18` | `soft @ 0.24` |
| Tinta `accent` da faixa de cabeçalho | `accent @ 0.14` | `accent @ 0.20` |
| Cores personalizadas de componentes | `C` | `C_light` (clarear ~15%) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. `viewBox = "0 0 {viewBox_w} {viewBox_h}"` derivado via §2 (5 camadas + 2 caminhos → 1040 × 548).
2. Cada cartão de camada em `(tier_x(i), 80)`, tamanho `172 × 380`, `rx=6`.
3. A faixa de cabeçalho da camada preenche `(tier_x(i), 80, 172, 40)` mais uma extensão de 10 px abaixo da faixa.
4. Exatamente **uma** camada focal; seu arco de entrada recebe automaticamente o estilo `focal`.
5. Os arcos de promoção são renderizados como Béziers cúbicas por cima de camadas adjacentes; âncoras em `(tier_cx(i), 80)` → `(tier_cx(i+1), 80)`, pontos de controle em y=0. Rótulo dentro do arco em `(arc_peak_x, 50)`, sem máscara.
6. A linha inferior de caminhos só está presente quando `len(paths) > 0`. Cartões em `y=476`, altura 56.
7. Cores personalizadas de componentes ≤ 2, além da camada focal. Nunca em setas.
8. Todas as setas de promoção + máscaras de rótulo são emitidas **antes** de qualquer retângulo de camada (regra de ordem Z — os cartões mascaram as pontas das linhas dentro dos cartões).
9. O texto do bucket e os valores de exemplo da camada focal são renderizados em `accent`; o restante permanece `muted`.
10. `rx=6` em todo cartão de camada e de caminho; `rx=2` nos chips de tag.

---

## 8. Antipadrões

- **Mais de uma camada focal** — o focal existe para marcar a superfície analítica central; >1 apaga o sinal.
- **Estilo cold em uma camada que não é de arquivo** — o visual tracejado de névoa (fog) é reservado para camadas de retenção/arquivo.
- **Setas de promoção bidirecionais** — as promoções sempre fluem da esquerda → direita. Fluxo reverso (ex.: um agregado escrevendo de volta no raw) é errado para este tipo; use outro diagrama.
- **Setas com cor personalizada** — os conectores são definidos pela topologia; a cor de uma camada nunca se espalha para suas arestas.
- **Cartões de caminho explicando a semântica das camadas** — os caminhos descrevem *métodos de escrita* (como os dados se movem entre as camadas), não o que cada camada contém. Se você se pegar escrevendo "Raw armazena …" em um cartão de caminho, esse conteúdo pertence aos campos da camada Raw.
- **Conteúdo de `example_label` ausente** — toda camada deve mostrar uma carga de exemplo concreta (linhas de pesquisa trimestral, registros de clientes, sinistros, …). Sem isso, o diagrama fica abstrato e deixa de justificar o espaço que ocupa.
- **Rótulo da seta de promoção mais longo que a máscara de rótulo do vão entre camadas** — mantenha os rótulos com ≤ 14 caracteres no papel `arrow-label` em maiúsculas. Verbos longos ("CALCULATE & SUMMARIZE") quebram o ritmo; encurte para "AGGREGATE" ou divida em dois diagramas.

---

## 9. Exemplos

- `assets/example-medallion.html` — claro mínimo (pesquisa trimestral NatStat: 5 camadas, 2 cartões de caminho, Aggregated como focal). Padrão da galeria.
- `assets/example-medallion-dark.html` — o mesmo, skin escuro.
- `assets/example-medallion-full.html` — o mesmo, com moldura de cartão editorial com subtítulo + cartões de resumo.

---

## 10. YAML de exemplo completo

O YAML em §1 é a definição **completa** de entradas do `example-medallion.html` distribuído. Toda coordenada no SVG desse arquivo pode ser derivada aplicando §2 a essas entradas. O mesmo YAML está incorporado como um comentário HTML no topo do `example-medallion.html`, para que a visualização do código-fonte mostre as entradas paramétricas imediatamente acima do SVG.
