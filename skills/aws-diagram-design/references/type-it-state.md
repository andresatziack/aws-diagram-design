# Estado atual de TI (IT current-state)

**Ideal para:** documentar o retrato do *antes* de uma proposta de modernização — o panorama legado de TI agrupado por fase ou departamento (Collection → Processing → Dissemination, ou Frontend / Backend / Storage, ou Survey → Analysts → Reports), com pontos de dor sinalizados, transferências baseadas em arquivos rotuladas (CSV / Excel / Email / Copy) e as ferramentas anteriores à plataforma visíveis. É o complemento de `type-dp-integration.md`: este tipo mostra a lacuna que uma proposta de plataforma de dados vai fechar.

Use quando os stakeholders precisam enxergar o atrito na configuração atual — scripts isolados, movimentação manual de arquivos, ausência de controle de versão, pontos únicos de falha — e o caminho deles até uma topologia de plataforma-alvo.

Este tipo é **paramétrico** — o esquema de entradas em §1 determina todas as coordenadas por meio das fórmulas em §2. O formato das regras espelha `type-dp-integration.md` (zonas + barras de rodapé transversais), `type-process.md` (conectores em ângulo reto arredondados) e `type-medallion.md` (sobrescrita de `color` por elemento), de modo que a regra focal, a sobrescrita de cor, o modo escuro e o checklist de reprodutibilidade sejam lidos de forma idêntica entre os tipos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
title:    "Current IT Landscape"
subtitle: "Data pipeline before the platform"
eyebrow:  "NatStat · Before the platform"

orientation: horizontal      # horizontal (default, zones L→R) | vertical (zones T→B)

zones:                       # 2..4 zones, ordered along the orientation axis
  - name: "COLLECTION"
    components:              # 1..5 components per zone
      - id: survey-solutions
        name: "Survey Solutions"
        sub:  "CAPI · PostgreSQL"
        icon: postgres               # any id from references/primitive-icons.md
        kind: standard               # standard | focal | external (external → dashed stroke)
      - { id: aspnet,    name: "ASP.NET Apps",    sub: "migration · admin portals", icon: server }
      - { id: civil-reg, name: "Civil Registry",  sub: "external · CRVS data",      icon: database, kind: external }
  - name: "PROCESSING"
    components:
      - { id: shared-drive,  name: "Shared Drive",     sub: "No version control · Windows file share", icon: file,      kind: focal }
      - { id: analyst-mach,  name: "Analyst Machines", sub: "SPSS · SAS · Stata · Excel",              icon: desktop }
      - { id: sql-server,    name: "SQL Server",       sub: "on-premises · core RDBMS",                icon: sqlserver, color: "#7a8c47" }  # custom olive
  - name: "DISSEMINATION"
    components:
      - { id: legacy-portal,   name: "LegacyPortal",      sub: "manual bottleneck",     icon: cloud,    kind: focal }
      - { id: natstat-website, name: "NatStat Website",   sub: "public · static pages", icon: internet }
      - { id: ministry,        name: "Ministry Partners", sub: "~6 ministries",         icon: users,    kind: external }

connectors:                   # ordered list; each links two component ids
  - { from: survey-solutions, to: shared-drive,   label: "CSV",     icon: csv,   style: link }
  - { from: aspnet,           to: shared-drive,   label: "EMAIL",   icon: file,  style: link }      # `mail` MISSING in catalog → falls back to `file`
  - { from: civil-reg,        to: shared-drive,   label: "EXCEL",   icon: excel, style: link, dashed: true }
  - { from: shared-drive,     to: analyst-mach,   label: "COPY",                 style: accent, dashed: true }
  - { from: analyst-mach,     to: sql-server,     label: "LOAD",                 style: neutral }
  - { from: analyst-mach,     to: legacy-portal,   label: "EXCEL",  icon: excel, style: accent }
  - { from: legacy-portal,    to: natstat-website, label: "WEB",                 style: neutral }
  - { from: natstat-website,  to: ministry,        label: "CSV DL", icon: csv,   style: link, dashed: true }

footer:                       # 0..3 optional full-canvas-width bars (cross-cutting concerns)
  - { name: "Identity Manager", sub: "Active Directory · LDAP · SSO", icon: active-directory }
  - { name: "Observability",    sub: "logs · metrics · alerts",       icon: monitoring }

legend:                       # auto-generated from styles used; user can override labels
  - { swatch: link,    label: "data flow" }
  - { swatch: accent,  label: "pain-point" }
  - { swatch: dashed,  label: "external" }
  - { swatch: focal,   label: "bottleneck" }

dark: false
```

**Semântica dos campos reservados:**

- `orientation` — `horizontal` (as zonas correm da esquerda para a direita, os componentes se empilham verticalmente dentro de cada zona) ou `vertical` (as zonas se empilham de cima para baixo, os componentes correm da esquerda para a direita dentro de cada zona).
- `zones[i].name` — rótulo curto em maiúsculas (≤ 14 caracteres). Renderizado no canto superior esquerdo da caixa da zona no papel `eyebrow` com letter-spacing de 0.14em, sobre uma interrupção mascarada com `paper` na borda da zona.
- `components[i][k].id` — slug globalmente único; referenciado por `connectors[].from/to`.
- `components[i][k].name` — papel `node-name` (o rótulo legível por humanos).
- `components[i][k].sub` — papel `sublabel` em 10px com cor `muted` (o sub-rótulo técnico; até 2 linhas via quebra automática quando a altura do componente cresce para 72).
- `components[i][k].icon` — qualquer id de `references/primitive-icons.md`. Se ausente → sem ícone, e o nome se desloca para a esquerda. (O catálogo tem 41 ícones; `mail` está ausente no momento — use `icon: file` como fallback para transferências por e-mail.)
- `components[i][k].kind` — `standard | focal | external`. `focal` aciona a paleta de destaque (§5); `external` muda para um traço tracejado 4-2 e tinta `muted` para sinalizar "isto está fora do nosso escopo."
- `components[i][k].color` — sobrescrita de cor opcional por componente (§4). Ignorada em `kind: focal` (`accent` prevalece).
- `connectors[k].from` / `connectors[k].to` — referem-se a um `id` de componente. Entre zonas, entre linhas, vertical na mesma zona e horizontal na mesma zona são todos válidos; o roteamento é escolhido por §3.
- `connectors[k].label` — texto curto em maiúsculas (≤ 8 caracteres). Papel `arrow-label` em 9px, peso 600.
- `connectors[k].icon` — ícone inline opcional à esquerda do texto. Mesmo catálogo dos ícones de componentes.
- `connectors[k].style` — `neutral | link | accent`. Determina a cor do traço + o marcador.
- `connectors[k].dashed` — `true | false`.
- `footer[k]` — barra transversal opcional. Ocupa toda a largura do canvas menos as margens. Nenhum conector é desenhado a partir de um rodapé.
- `legend[k].swatch` — `link | accent | dashed | focal | neutral`. Curadoria automática com base no que o diagrama realmente usa; o usuário pode reordenar ou renomear.

---

## 2. Fórmulas de layout — geometria determinística

```
# Horizontal orientation (default)
left_pad        = 16
right_pad       = 16
zone_gap        = 20
zone_y          = 52
zone_h          = 360
n_zones         = len(zones)

# Zone widths: base 200 + 24 per component to give vertical room for icons + 2-line subs.
# In the canonical example (3 / 3 / 3 components) the replication used 256 / 360 / 272 —
# the formula approximates that with hand-picked widths in the worked YAML (§10).
zone_w(i)       = base + n_components_i * comp_slack             # base ≈ 200, slack ≈ 24
viewBox_w       = left_pad + Σ zone_w(i) + (n_zones-1) * zone_gap + right_pad

# Component placement within zone i
comp_pad_x      = 20                                              # x-inset from zone border
comp_h          = 56                                              # 68 for focal (2-line sub), 72 if both sub lines present
comp_gap        = 32
comp_y(i, k)    = zone_y + 28 + k * (comp_h + comp_gap)

# Component centerlines (used for connector routing)
comp_x(i)       = zone_x(i) + comp_pad_x
comp_w(i)       = zone_w(i) - 2 * comp_pad_x
comp_cx(i)      = comp_x(i) + comp_w(i)/2
comp_cy(i, k)   = comp_y(i, k) + comp_h/2

# Footer bars (if present)
footer_bar_h    = 56
footer_gap      = 8
footer_top      = zone_y + zone_h + 24
footer_y(k)     = footer_top + k * (footer_bar_h + footer_gap)
footer_bottom   = footer_top + N_footer * (footer_bar_h + footer_gap) - footer_gap

# Total canvas height
legend_block_h  = 40
content_bottom  = N_footer > 0 ? footer_bottom : zone_y + zone_h
viewBox_h       = content_bottom + legend_block_h + 24
```

### 2.1 Fundo e moldura da zona

Preenchimento sólido `paper` em todo o `viewBox`. Sem padrão de pontos. Cada caixa de zona:

```svg
<rect x="zone_x(i)" y="zone_y" width="zone_w(i)" height="zone_h"
      fill="{ink @ 0.02}" stroke="{ink @ 0.10}" stroke-width="0.8" rx="8"/>
<!-- paper-masked break for the zone label -->
<rect x="zone_x(i)+20" y="zone_y-8" width="{label_w}" height="16" fill="{paper}"/>
<text x="zone_x(i)+24" y="zone_y+4" fill="{ink @ 0.40}"
      font-family="{eyebrow}" letter-spacing="0.14em">{name}</text>
```

### 2.2 Caixa do componente

Três tipos visuais:

| `kind` | Preenchimento | Traço | Largura do traço | Tracejado | Tinta do nome | Tinta do sub |
| --- | --- | --- | --- | --- | --- | --- |
| `standard` | `#FFFFFF` | `ink` | 1 | — | `ink` | `muted` |
| `focal` | `accent @ 0.07` | `accent` | 1.4 | — | `ink` | `accent` (linha 1) + `muted` (linha 2) |
| `external` | `#FFFFFF` | `muted` | 1 | `4,3` | `ink` | `muted` |

**Posicionamento do ícone** (24×24, monocromático via `currentColor` — veja `references/primitive-icons.md`):

```svg
<g transform="translate(comp_x + 12, comp_y + (comp_h - 24)/2)" color="{ink_for_kind}">
  <use href="#icon-{name}"/>            <!-- or inline the SVG path from the catalog -->
</g>
```

O ícone ocupa 24 × 24 → pegada horizontal total de 36 px com o padding esquerdo de 12 px. A linha de base do nome e do sub-rótulo se desloca 40 px para a direita.

**Linhas de base do nome + sub** (alinhadas à esquerda, com o ícone à esquerda):

```
name_x = comp_x + 44
name_y = comp_y + (comp_h/2) - 2
sub_y  = comp_y + (comp_h/2) + 14
```

### 2.3 Geometria dos conectores (§3 contém as regras de roteamento)

```
src_right  = comp_x(i_src) + comp_w(i_src)
src_left   = comp_x(i_src)
src_top    = comp_y(i_src, k_src)
src_bot    = src_top + comp_h_src
src_cy     = src_top + comp_h_src/2

dst_left   = comp_x(i_dst)
dst_right  = comp_x(i_dst) + comp_w(i_dst)
dst_top    = comp_y(i_dst, k_dst)
dst_bot    = dst_top + comp_h_dst
dst_cx     = comp_cx(i_dst)
dst_cy     = dst_top + comp_h_dst/2

# Corridor x for cross-zone H+Q+V routing
corridor_x = dst_cx                            # land arrow on dst horizontal center, enter via top/bot
```

### 2.4 Barra de rodapé

```svg
<rect x="left_pad" y="footer_y(k)" width="viewBox_w - 2*left_pad" height="footer_bar_h"
      fill="{ink @ 0.03}" stroke="{ink @ 0.18}" stroke-width="0.8" rx="8"/>
<g transform="translate(left_pad + 20, footer_y(k) + (footer_bar_h - 24)/2)" color="{ink}">
  <use href="#icon-{name}"/>
</g>
<text x="left_pad + 56" y="footer_y(k) + 24" font-family="{node-name}" font-size="14" fill="{ink}">{name}</text>
<text x="left_pad + 56" y="footer_y(k) + 40" font-family="{sublabel}" font-size="12" fill="{muted}">{sub}</text>
```

As barras de rodapé são serviços de toda a camada. **Nenhum conector sai delas.** Elas ficam visualmente abaixo das zonas e permitem que o leitor veja as preocupações transversais de relance.

### 2.5 Faixa da legenda

Divisor de linha fina (hairline) em `y = content_bottom + 16`, depois uma linha de amostras + rótulos em `y = content_bottom + 36`. Apenas os estilos realmente usados por `connectors[]` (mais `focal` e `external`, se esses tipos de componente estiverem presentes) aparecem na legenda.

---

## 3. Regras de conectores (obrigatórias)

### 3.1 Formato do caminho — Q-bezier em ângulo reto arredondado, r = 8

Reutilizado literalmente de `type-process.md` §3.1. Sem diagonais — nunca.

```svg
<!-- Same zone, adjacent component (same vertical column): single vertical line -->
<line x1="{src_cx}" y1="{src_bot}" x2="{dst_cx}" y2="{dst_top}"
      stroke="…" stroke-width="…" marker-end="…"/>

<!-- Cross-zone or cross-row: exit right → H → Q-bend → V → enter top (or bottom) -->
<path d="M {src_right},{src_cy}  H {dst_cx - 8}  Q {dst_cx},{src_cy} {dst_cx},{src_cy ± 8}  V {dst_top_or_bottom}"
      fill="none" stroke="…" stroke-width="…" marker-end="…"/>
```

- Use `{src_cy + 8}` e `V {dst_top}` quando o destino estiver ABAIXO da origem.
- Use `{src_cy − 8}` e `V {dst_bottom}` quando o destino estiver ACIMA da origem.

### 3.2 Lados de saída / entrada (configuráveis; padrões abaixo)

| Topologia | Lado de saída padrão da origem | Lado de entrada padrão do destino |
| --- | --- | --- |
| Mesma zona, destino abaixo | inferior | superior |
| Mesma zona, destino acima | superior | inferior |
| Entre zonas, fluxo horizontal | direito | superior (ou inferior, o que estiver mais próximo de src_cy) |
| Diagrama de orientação vertical | inferior | superior |

Um conector pode sobrescrever via `connectors[k].from_side` / `connectors[k].to_side` (`top | right | bottom | left`). **Referências para trás** (da direita para a esquerda na orientação horizontal, para cima na orientação vertical) só são permitidas quando pelo menos uma extremidade tem `kind: external`, e devem ser `dashed: true`.

### 3.3 Os marcadores DEVEM tocar o retângulo de destino

O último comando do caminho termina na borda do retângulo de destino (`V {dst_top}` ou `H {dst_left}`), **não** no centroide. Após aplicar `refX=7` no marcador, o triângulo fica rente à borda. Parar a linha antes da borda — ou levá-la até o centroide e enterrar a ponta da seta dentro do retângulo — é uma falha grave.

### 3.4 Estilo → traço + marcador

| `style` | Cor do traço | Largura do traço | Marcador |
| --- | --- | --- | --- |
| `neutral` | `muted` | 1.0 | `url(#arrow)` |
| `link` | `link` | 1.2 | `url(#arrow-link)` |
| `accent` | `accent` | 1.4 | `url(#arrow-accent)` |

Adicione `stroke-dasharray="4 3"` quando `dashed: true`.

**Bloco defs** (sempre emita os três):

```svg
<defs>
  <marker id="arrow"        markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="{muted}"/></marker>
  <marker id="arrow-link"   markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="{link}"/></marker>
  <marker id="arrow-accent" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto"><polygon points="0 0, 8 3, 0 6" fill="{accent}"/></marker>
</defs>
```

### 3.5 Rótulo do conector = ícone inline + texto, posicionado no INÍCIO do conector com uma margem perpendicular

O rótulo fica **perto da extremidade de origem** do conector (não no meio do segmento) e é **deslocado perpendicularmente à linha** para nunca se sobrepor ao traço. O ícone (quando `icon:` está definido) fica dentro da máscara com preenchimento `paper` do rótulo, à esquerda do texto.

```svg
<g transform="translate({label_cx}, {label_cy})">
  <rect x="-{w/2}" y="-9" width="{w}" height="18" rx="3" fill="{paper}" stroke="none"/>
  <use href="#icon-{icon}" x="-{w/2 + 4}" y="-6" width="12" height="12" color="{stroke_color}"/>     <!-- if icon set -->
  <text x="{icon ? (-(w/2) + 22) : 0}" y="3" text-anchor="{icon ? 'start' : 'middle'}"
        font-family="{arrow-label}" font-size="9" font-weight="600"
        letter-spacing="0.08em" fill="{stroke_color}">{label}</text>
</g>
```

**Fórmulas de posicionamento** (a caixa do rótulo tem 18px de altura × `w` de largura; centralizada em `{label_cx, label_cy}`):

| Segmento saindo da origem | `label_cx` | `label_cy` | Efeito |
| --- | --- | --- | --- |
| Horizontal (saída à direita) | `src_right + 6 + w/2` | `src_cy − 14` | O rótulo fica 6 px depois da origem, 5 px acima da linha |
| Horizontal (saída à esquerda, para trás) | `src_left − 6 − w/2` | `src_cy − 14` | O rótulo fica 6 px antes da origem, 5 px acima da linha |
| Vertical (saída inferior) | `src_cx + 6 + w/2` | `src_bot + 14` | O rótulo fica 6 px à direita da linha, 5 px abaixo da borda da origem |
| Vertical (saída superior, para trás) | `src_cx + 6 + w/2` | `src_top − 14` | O rótulo fica 6 px à direita da linha, 5 px acima da borda da origem |

Para rotas H+Q+V entre zonas, o rótulo se vincula ao segmento **horizontal**, já que esse segmento está ancorado na origem. Posicione o rótulo no início desse trecho horizontal — nunca na curva Q nem na cauda vertical.

- `w = text_w + (icon ? 30 : 12)` — ajuste automático.
- O `fill` da máscara resolve para `paper` no modo claro e `ink` no modo escuro. A máscara é mantida como um respiro de segurança — mesmo que o rótulo não fique mais sobre a linha, ele pode tocar fundos de zona e preenchimentos de componentes, e a máscara preserva o contraste.
- `stroke_color` segue §3.4 (texto + ícone herdam a cor `accent` / `link` / neutral do conector).

### 3.6 Ordem Z

Todos os conectores (paths + lines + rótulos) são emitidos ANTES de qualquer rect de componente, para que os preenchimentos dos nós mascarem as pontas das linhas. O rótulo do conector é a exceção — ele é desenhado DEPOIS de sua linha para que a máscara fique por cima.

---

## 4. Sobrescrita de cor de componente (por componente `color: "#hex"`)

Por componente, mesmo formato de todos os outros tipos paramétricos desta skill.

| Elemento | Claro | Escuro |
| --- | --- | --- |
| Preenchimento do contêiner | `rgba(C, 0.06)` | `rgba(C_light, 0.10)` |
| Traço do contêiner | `rgba(C, 0.45)` (largura 1) | `rgba(C_light, 0.55)` |
| Texto do nome do componente | `C` | `C_light` |
| Glifo do ícone | herda `ink` via `currentColor` (inalterado) | herda `ink` (inalterado) |
| Sub-rótulo | `muted` (inalterado) | `muted` (inalterado) |
| Conectores que tocam este componente | **inalterados** — determinados pela topologia | **inalterados** |

`C_light` = o mesmo hex clareado ~15 % para contraste no modo escuro (ex.: `#7a8c47` → `#9aac67`, `#b85450` → `#d97a78`).

**Regras:**
- **Nunca em componentes focais.** `kind: focal` sempre renderiza em `accent`; `color` é ignorado silenciosamente.
- **Nunca em conectores.** O estilo do conector é determinado pela topologia; se você quer uma aresta colorida, escolha `style: accent` / `link` / `neutral`, não uma cor de componente.
- **Limite: ≤ 3 componentes com cor personalizada por diagrama** (além dos componentes focais). Acima de 3, o sinal visual se fragmenta.

**Paleta recomendada entre tipos** (a mesma de `type-medallion.md` / `type-process.md` / `type-dp-integration.md` / `type-dp-security-matrix.md`):

- `#b85450` vermelho-ferrugem — segurança / governança / ponto de dor que não é focal
- `#5a7d9a` azul-ardósia — observabilidade / qualidade / portão de monitoramento
- `#7a8c47` verde-oliva — sistema sobrevivente (a única ferramenta que a nova plataforma mantém)
- `#c9a23a` amarelo quente — sandbox / dev / rascunho
- `#8c6d3f` marrom quente — arquivo / frio / DR

---

## 5. Regra focal

- Componentes `kind: focal`: **≤ 2 por diagrama** (zero também é válido para diagramas sem um único ponto de dor dominante).
- Estilização automática: traço `accent` 1.4, preenchimento tingido de `accent` a 7 %, `node-name` em `ink` negrito, `sublabel` da linha 1 em `accent`.
- Qualquer conector com uma extremidade focal é renderizado automaticamente em `style: accent`; o `style:` do YAML é ignorado.
- `color: "#hex"` personalizado em um componente focal é ignorado silenciosamente — `accent` sempre prevalece.

Se o seu diagrama precisa de mais de 2 componentes focais, você juntou duas narrativas. Divida: um diagrama de "pontos de dor da coleta" + um diagrama de "pontos de dor da disseminação".

---

## 6. Modo escuro

| Papel | Claro | Escuro |
| --- | --- | --- |
| paper | `paper` | `ink` |
| ink | `ink` | `paper` |
| muted | `muted` | `muted` |
| accent | `accent` | `accent` |
| link | `link` | `link` |
| fundo da zona | `ink @ 0.02` | `paper @ 0.04` |
| borda da zona | `ink @ 0.10` | `paper @ 0.14` |
| preenchimento do componente padrão | `#FFFFFF` | `paper @ 0.04` |
| traço do componente padrão | `ink` | `paper @ 0.32` |
| preenchimento focal | `accent @ 0.07` | `accent @ 0.12` |
| traço focal | `accent` | `accent` |
| traço externo | `muted` (tracejado) | `muted` (tracejado) |
| preenchimento do rodapé | `ink @ 0.03` | `paper @ 0.05` |
| traço do rodapé | `ink @ 0.18` | `paper @ 0.20` |
| preenchimento da máscara do rótulo | `paper` | `ink` |
| componentes com cor personalizada | `C` | `C_light` (≈ +15 %) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. Eyebrow + título + subtítulo presentes nas posições y canônicas (24 / 36 / 52); padding do corpo de 32 px.
2. 2..4 zonas; cada uma tem seu rótulo em maiúsculas no canto superior esquerdo da caixa da zona, sobre uma interrupção mascarada com `paper` na borda.
3. Todo componente tem `id`, `name`. `sub`, `icon`, `kind`, `color` são opcionais.
4. ≤ 2 componentes com `kind: focal`; estilização focal aplicada automaticamente (preenchimento `accent` a 7 %, traço `accent` 1.4, sub da linha 1 em itálico).
5. Todo conector sai pela direita (ou por baixo) da origem e entra por cima (ou pela esquerda) do destino; Q-bezier em ângulo reto arredondado `r=8` em toda curva; o triângulo do marcador toca visivelmente a borda do retângulo de destino.
6. Os rótulos dos conectores ficam no **início** do conector (não no meio do segmento) e são deslocados **perpendicularmente** à linha (vão de 5 px acima para segmentos horizontais, vão de 6 px à direita para segmentos verticais) — nunca sobrepondo o traço. Máscara com preenchimento `paper` mantida atrás do texto; o ícone (quando `icon:` está definido) fica à esquerda do texto dentro da mesma máscara.
7. ≤ 3 componentes com cor personalizada; nenhum em focal.
8. ≤ 3 barras de rodapé; cada uma ocupa `viewBox_w − 2*left_pad`; nenhum conector sai de nenhum rodapé.
9. Legenda na parte inferior: separador de linha fina (hairline) + uma amostra por estilo realmente usado.
10. `arrow-label` para rótulos de conectores, `eyebrow` para o eyebrow da página e rótulos de zona, `title` para o título da página, `node-name` para o subtítulo e os nomes dos componentes, e `sublabel` para os sub-rótulos técnicos.
11. Marcadores `#arrow` / `#arrow-link` / `#arrow-accent` definidos uma única vez em `<defs>`; nenhuma definição de marcador inline.
12. Variante escura: resolva cada token semântico por meio de seu valor no modo escuro; as cores personalizadas são clareadas ~15 %.

---

## 8. Antipadrões

- **Setas diagonais.** A replicação do NatStat tem uma (analyst → LegacyPortal). O novo tipo a proíbe — sempre Q-bezier em ângulo reto arredondado.
- **Marcador sem tocar o alvo.** O caminho termina no centroide ou para antes da borda.
- **Rótulos de conector com `text` inline sem um rect de máscara.** A linha do conector pode vazar através do texto, que se torna ilegível.
- **Rótulos sobre a linha do conector, no meio do segmento.** Os rótulos pertencem ao *início* do conector com uma margem perpendicular (veja §3.5) — enterrá-los no meio da linha esconde a direção da origem ao destino e força o olho do leitor a brigar com a máscara.
- **Pequenos selos de texto como ícones.** A fonte usa selos de 7 px `DB` / `APP` / `EXT`; este tipo usa ícones reais de 24 px do catálogo. Selos de texto só são aceitáveis como texto do rótulo, não como "ícone" do componente.
- **Cor personalizada em um componente focal.** Focal sempre prevalece; o `color` definido pelo usuário é ignorado silenciosamente em `kind: focal`.
- **Barra de rodapé ligada a um único componente.** Rodapé = preocupação transversal de toda a camada; um conector de um rodapé para um componente específico é um erro de categoria (use o padrão de linha AUTH de `type-dp-integration.md` somente quando o serviço do rodapé realmente autentica *todos* os componentes, e mesmo assim a linha chega à borda inferior da zona, não a uma ferramenta específica).
- **> 16 componentes no total ou > 5 por zona.** Limite de densidade; divida em dois diagramas.
- **Misturar orientações em um mesmo diagrama.** Escolha uma — `horizontal` ou `vertical` — e aplique-a a todas as zonas.
- **Usar `kind: focal` para sinalizar tudo o que é problemático.** Focal existe para ≤ 2 pontos de dor narrativos; para "isto é ruim, mas não é ruim de manchete", use `color: "#b85450"` vermelho-ferrugem.

---

## 9. Exemplos

- `assets/example-it-state.html` — mínimo claro (canônico NatStat: 3 zonas, 9 componentes, 8 conectores, 0 barras de rodapé, SQL Server tingido de oliva). Padrão da galeria.
- `assets/example-it-state-dark.html` — o mesmo, skin (tema) escura.
- `assets/example-it-state-full.html` — o mesmo, moldura de cartão editorial com cartões de resumo.
- `assets/example-it-state-extended.html` — exercita a sobrescrita de cor de §4 + barras de rodapé: 2 barras de rodapé (Identity Manager + Observability) abaixo das zonas, terceira cor personalizada em Analyst Machines (azul-ardósia, preocupação com qualidade de dados).
- `assets/example-it-state-extended-dark.html` — padrão estendido, skin (tema) escura.

---

## 10. YAML trabalhado — entradas completas para `example-it-state.html`

As entradas completas que mapeiam para o exemplo canônico distribuído. Toda coordenada naquele SVG pode ser derivada aplicando §2 a estas entradas.

```yaml
title:    "Current IT Landscape"
subtitle: "Data pipeline before the platform"
eyebrow:  "NatStat · Before the platform"

orientation: horizontal

zones:
  - name: "COLLECTION"
    components:
      - { id: survey-solutions, name: "Survey Solutions", sub: "CAPI · PostgreSQL",          icon: postgres }
      - { id: aspnet,           name: "ASP.NET Apps",     sub: "migration · admin portals", icon: server   }
      - { id: civil-reg,        name: "Civil Registry",   sub: "external · CRVS data",      icon: database, kind: external }
  - name: "PROCESSING"
    components:
      - { id: shared-drive,  name: "Shared Drive",     sub: "No version control · Windows file share", icon: file,      kind: focal }
      - { id: analyst-mach,  name: "Analyst Machines", sub: "SPSS · SAS · Stata · Excel",              icon: desktop }
      - { id: sql-server,    name: "SQL Server",       sub: "on-premises · core RDBMS",                icon: sqlserver, color: "#7a8c47" }
  - name: "DISSEMINATION"
    components:
      - { id: legacy-portal,   name: "LegacyPortal",      sub: "manual bottleneck",     icon: cloud,    kind: focal }
      - { id: natstat-website, name: "NatStat Website",   sub: "public · static pages", icon: internet }
      - { id: ministry,        name: "Ministry Partners", sub: "~6 ministries",         icon: users,    kind: external }

connectors:
  - { from: survey-solutions, to: shared-drive,   label: "CSV",    icon: csv,   style: link }
  - { from: aspnet,           to: shared-drive,   label: "EMAIL",  icon: file,  style: link }
  - { from: civil-reg,        to: shared-drive,   label: "EXCEL",  icon: excel, style: link, dashed: true }
  - { from: shared-drive,     to: analyst-mach,   label: "COPY",                style: accent, dashed: true }
  - { from: analyst-mach,     to: sql-server,     label: "LOAD",                style: neutral }
  - { from: analyst-mach,     to: legacy-portal,   label: "EXCEL",  icon: excel, style: accent }
  - { from: legacy-portal,    to: natstat-website, label: "WEB",                 style: neutral }
  - { from: natstat-website,  to: ministry,        label: "CSV DL", icon: csv,   style: link, dashed: true }

dark: false
```

### 10.1 O que este YAML comprova

- `n_zones = 3`, componentes por zona `= [3, 3, 3]`, quantidade de cores personalizadas = 1 (SQL Server), quantidade de focais = 2 (Shared Drive, LegacyPortal), quantidade de externos = 2 (Civil Registry, Ministry Partners).
- Larguras das zonas no canônico: 256 / 360 / 272 ⇒ `viewBox_w = 16 + 256 + 20 + 360 + 20 + 272 + 16 = 960` ✓
- `viewBox_h = 52 + 360 + 40 + 24 = 500` (sem barras de rodapé) ✓
- Shared Drive (focal) na zona 2, linha 0: `x = 340, y = 80, w = 264, h = 68` (o focal se estende para 68 para caber o sub de 2 linhas) ✓
- LegacyPortal (focal) na zona 3, linha 0: `x = 704, y = 80, w = 208, h = 60` ✓
- SQL Server (oliva personalizado) na zona 2, linha 2: preenchimento do contêiner `rgba(122,140,71,0.06)`, traço `rgba(122,140,71,0.45)`, texto do nome `#7a8c47` ✓
- Os conectores 4, 5 (dentro da zona 2) e 7, 8 (dentro da zona 3) são elementos `<line>` verticais simples. Os conectores entre zonas seguem rotas compatíveis com as regras (veja SKILL.md §6 regras 4 e 5):
  - **Os três conectores do lado Survey → Shared Drive (C1 / C2 / C3) entram pela borda ESQUERDA do Shared Drive.** Uma entrada pela borda superior empurraria o corpo do marcador (7 px para trás ao longo do percurso, dado `refX = 7`) para *dentro* da caixa de destino, onde a máscara de preenchimento `paper` da caixa o esconde — apenas uma ponta de 1 pixel apareceria acima do traço. Entrar pela borda esquerda com um caminho indo para a direita mantém o corpo fora da caixa e a seta visível (~7 px mostrados à esquerda da borda da caixa). Os três pontos de conexão na borda esquerda são distribuídos (fan-out) em **y = 108 / 124 / 140** (espaçamento de 16 px, bem acima do mínimo de 12 px da regra 4).
  - **C1** (Survey → Shared Drive): o y de origem coincide com o y de chegada: um único horizontal `M 252,108 H 340`. Nenhuma curva necessária.
  - **C2** (ASP.NET → Shared Drive) desvia para cima pelo fundo da zona 2 — vertical em `x = 316` (afastado da borda esquerda do Shared Drive em `x = 340`): `H 308 Q 316,196 316,188 V 132 Q 316,124 324,124 H 340`. Chega em `(340, 124)`.
  - **C3** (Civil Registry → Shared Drive) desvia para cima pelo fundo da zona 2 — vertical em `x = 332` (afastado de Analyst Machines, que começa em `x = 340`): `H 324 Q 332,284 332,276 V 148 Q 332,140 340,140`. Chega em `(340, 140)` por meio de uma curva Q final (nenhum H final necessário).
  - **C6** (Analyst Machines → LegacyPortal) não pode usar um H+Q+V direto para a borda esquerda do LegacyPortal — Analyst Machines e LegacyPortal estão em linhas diferentes, e o horizontal direto em `y = 268` cruzaria NatStat Website. Ele desvia pelo vão entre zonas e passa **por cima** do LegacyPortal: `H 654 Q 662,268 662,260 V 72 Q 662,64 670,64 H 800 Q 808,64 808,72 V 80` — vertical em `x = 662` (no vão entre zonas), horizontal em `y = 64` (acima do topo do LegacyPortal), depois descendo até o centro superior do LegacyPortal. O caminho entra no LegacyPortal **por cima** descendo, então o corpo da seta fica acima da caixa (visível) e apenas a ponta de 1 px entra na caixa.

**Regra prática de visibilidade do marcador:** com o marcador de seta padrão (`markerWidth = 8`, `refX = 7`), o corpo da seta se estende 7 px *para trás* ao longo da direção do caminho a partir do ponto final. Para que a seta permaneça visível, essa cauda de 7 px precisa ficar *fora* da caixa de destino. Tradução:
- Entrar por uma **borda SUPERIOR SUBINDO** (direção do caminho para cima, caixa abaixo) → corpo dentro da caixa, **apenas 1 px visível. Evite isto.**
- Entrar por uma **borda SUPERIOR DESCENDO** (direção do caminho para baixo, caixa abaixo) → corpo acima da caixa, ~7 px visíveis. ✓
- Entrar por uma **borda ESQUERDA indo para a DIREITA** → corpo à esquerda da caixa, ~7 px visíveis. ✓
- Entrar por uma **borda DIREITA indo para a ESQUERDA** → corpo à direita da caixa, ~7 px visíveis. ✓
- Entrar por uma **borda INFERIOR DESCENDO** (caixa acima) → corpo dentro da caixa, **apenas 1 px visível. Evite isto.**
- Entrar por uma **borda INFERIOR SUBINDO** (caixa acima) → corpo abaixo da caixa, ~7 px visíveis. ✓

Quando a linha de origem coincide com a faixa de y da linha de destino (ex.: Survey em y=108 com Shared Drive em y=80–148), prefira a entrada pela **borda lateral** — um único caminho horizontal com uma seta totalmente visível. Quando a linha de origem está deslocada, desvie pelo fundo da zona mais próxima do destino para entrar por uma borda lateral em vez de se aproximar de uma borda superior/inferior pelo lado errado.

O exemplo estendido (§9 linha 4) demonstra barras de rodapé + uma terceira cor personalizada e comprova que `viewBox_h` cresce corretamente quando `N_footer > 0`.
