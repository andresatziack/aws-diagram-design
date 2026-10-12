# Integração de DP (DP integration)

**Ideal para:** a topologia de integração de uma plataforma de dados — quais sistemas de origem se conectam na entrada, quais superfícies de consumo se conectam na saída e qual protocolo cada um utiliza. Layout hub-and-spoke envolto em uma camada explícita de **Data platform** (plataforma de dados); sem eixo de tempo/fase.

Use quando a pergunta for *"quais superfícies esta plataforma expõe e por qual meio (wire)?"* em vez de *"como os dados se movem pelas fases?"*.

Este tipo é **paramétrico** — assim como `type-high-level.md`, toda coordenada é derivada de um pequeno esquema de entradas. Duas gerações a partir das mesmas entradas devem produzir SVGs visualmente idênticos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
sources:                            # left column, 0..6 nodes
  - { name: "Databases",  type: "db",        subtitle: "SQL · MariaDB",
      connects_to: [{to: "NiFi", label: "JDBC"},
                    {to: "Trino", label: "FEDERATE", style: "federated"}] }
  - { name: "SFTP drops", type: "sftp",      subtitle: "scheduled pulls",
      connects_to: [{to: "NiFi", label: "SFTP"}] }
  - { name: "Email",      type: "mail",      subtitle: "IMAP attachments",
      connects_to: [{to: "NiFi", label: "IMAP"}] }
  - { name: "IBM legacy", type: "mainframe", subtitle: "file export",
      connects_to: [{to: "NiFi", label: "FILE"}] }

platform:
  name: "DATA PLATFORM"             # zone label (paper-masked top border)
  rows:                             # ordered top→bottom; each is bar or row
    - { kind: bar, name: "Trino",   icon: trino,   subtitle: "federated query · push-down",
        role: "SQL", focal: true }
    - { kind: row, nodes: [
        { name: "Apache NiFi",  icon: nifi,    role: "INGEST",   subtitle: "flow-based ETL" },
        { name: "MinIO",        icon: minio,   role: "STORE",    subtitle: "S3 object store · medallion", focal: true },
        { name: "JupyterLab",   icon: jupyter, role: "NOTEBOOK", subtitle: "Python · R · pandas" }
      ]}
    - { kind: bar, name: "Apache Airflow", icon: airflow,
        subtitle: "scheduler · DAG triggers · backfill", role: "DAG" }

consumers:                          # right column, 0..6 nodes
  - { name: "Desktop apps",   type: "monitor", subtitle: "SPSS · SAS · Stata",
      connects_from: [{from: "Trino", label: "ODBC"}] }
  - { name: "BI & reports",   type: "chart",   subtitle: "Tableau · Power BI",
      connects_from: [{from: "Trino", label: "JDBC"}] }
  - { name: "Public website", type: "globe",   subtitle: "NatStat portal",
      connects_from: [{from: "Trino", label: "HTTPS"}] }
  - { name: "API gateway",    type: "api",     subtitle: "3rd-party / OAuth2",
      connects_from: [{from: "Trino", label: "REST"}] }

footer:                             # 0..N cross-cutting bars stacked below zone (full-canvas width)
  - { name: "Active Directory", icon: key,        subtitle: "LDAP · SSO · group RBAC",
      color: "#b85450" }            # tinted red to flag the security concern
  # additional footer nodes (Observability, Backup, …) stack below this one

internal_connections:               # explicit platform-component edges
  - { from: "NiFi",        to: "MinIO",      style: "primary",   label: "WRITE" }
  - { from: "MinIO",       to: "JupyterLab", style: "secondary", label: "READ"  }
  - { from: "MinIO",       to: "Trino",      style: "secondary" }
  - { from: "JupyterLab",  to: "Trino",      style: "secondary", dashed: true }
  - { from: "Airflow",     to: ["Apache NiFi", "MinIO", "JupyterLab"], style: "trigger" }

focal_accent: "#EC7211"             # one color for all focal components (default = SKILL accent)
dark: false
```

**Valores reservados de `kind` para `platform.rows`:**
- `bar` — faixa com a largura total da zona. Altura padrão de 44 px (barras focais recebem 56 px). Campos obrigatórios: `name`, `icon`. Opcionais: `subtitle`, `role`, `color`, `focal`.
- `row` — N nós distribuídos uniformemente pela largura da zona. Obrigatório: lista `nodes`. Cada nó tem `name`, `icon` e, opcionalmente, `role`, `subtitle`, `color`, `focal`.

**Valores de `type` de origem/consumidor** → mapeamento de ícones (estende `references/primitive-icons.md`):
- `db` → cilindro, `sftp` → pasta com seta, `mail` → envelope, `mainframe` → servidor com grades de ventilação
- `monitor` → tela de desktop, `chart` → gráfico de barras, `globe` → globo, `api` → chaves
- `key` → chave + argola (identidade / IDP)
- Qualquer nome de ícone explícito em `primitive-icons.md` também é aceito.

**`color: "#hex"` por componente** é opcional em qualquer nó, barra ou entrada de rodapé. Veja §4.

---

## 2. Fórmulas de layout — geometria determinística

```
# Canvas
viewBox_w        = 1200
n_sources        = len(sources)
n_consumers      = len(consumers)
n_footer         = len(footer)

# Side columns (sources left, consumers right)
col_top          = 92
col_node_h       = 64
col_gap          = 24                    # stride = col_node_h + col_gap = 88
col_h_min        = 336                   # default fits 4 sources (4 * 88 - 24)
col_h            = max(col_h_min, max(n_sources, n_consumers) * 88 - 24)
left_x           = 40
left_w           = 160
right_x          = 1000
right_w          = 160
col_node_y(k)    = col_top + k * 88
col_node_cy(k)   = col_node_y(k) + col_node_h/2     # 124, 212, 300, 388 by default

# Platform zone
zone_x           = 260
zone_w           = 696
zone_y           = 72
zone_h           = col_h                            # zone always matches column height
zone_cx          = zone_x + zone_w/2                # 608
zone_pad_x       = 16                               # inside left/right padding for bars
zone_label_y     = zone_y + 3                       # paper-masked label across top border

# Footer bars (below zone — each cross-cutting concern is a full-width bar)
footer_top       = zone_y + zone_h + 52             # 52-px gap below zone
footer_bar_h     = 56
footer_bar_x     = 40                               # aligned with source column left edge
footer_bar_w     = viewBox_w - 80                   # = 1120 — spans from source col left to consumer col right
footer_gap       = 8
footer_y(k)      = footer_top + k * (footer_bar_h + footer_gap)
footer_bottom    = footer_top + n_footer * (footer_bar_h + footer_gap) - footer_gap

viewBox_h        = max(600, footer_bottom + 84)     # 84 reserved for legend

# Platform.rows allocation inside zone
bar_h_focal      = 56
bar_h_default    = 44
row_h            = 72
row_gap          = 16
```

### 2.1 Posicionamento das linhas (algoritmo de cursor)

Aloque cada entrada de `platform.rows` de cima para baixo. A única `row` (ou a primeira linha, quando N>1) é ancorada na linha 2 das colunas laterais para que seus conectores permaneçam horizontais:

```
primary_row_idx  = index of first kind=row in platform.rows
primary_row_top  = col_node_y(1) - (row_h - col_node_h)/2     # 176 by default
                                                              # 4-px nudge so cy aligns with side row 2

# Place rows above primary
y = primary_row_top
for entry in platform.rows[:primary_row_idx] reversed:
    y -= row_gap
    entry.h     = bar_h_focal if (entry.kind == bar and entry.focal) else bar_h_default
    y          -= entry.h
    entry.y_top = y                                            # Trino bar lands at y=104

# Place primary row
platform.rows[primary_row_idx].y_top = primary_row_top         # NiFi/MinIO/Jupyter at y=176
platform.rows[primary_row_idx].h     = row_h

# Place rows below primary
y = primary_row_top + row_h
for entry in platform.rows[primary_row_idx+1:]:
    y          += row_gap
    entry.y_top = y
    entry.h     = bar_h_focal if (entry.kind == bar and entry.focal) else bar_h_default
    y          += entry.h

# Constraint: y <= zone_y + zone_h
```

Isso produz o layout canônico para o formato padrão (barra superior / linha de 3 nós / barra inferior): Trino em `y=104 h=56`, linha primária em `y=176 h=72`, Airflow em `y=324 h=44`. A base do Airflow fica em y=368 (40 px de folga até a base da zona em y=408). **Observação**: no layout canônico há um espaço de 76 px entre a base da linha primária (y=248) e o topo do Airflow (y=324). Esse espaço é intencional — visualmente, o Airflow fica na mesma faixa de y que a linha 4 de origens/consumidores (cy=388), de modo que é lido como irmão da linha inferior das colunas laterais.

### 2.2 Posicionamento dos nós dentro de uma entrada `row`

```
N            = len(row.nodes)
node_w       = (zone_w - 2*zone_pad_x - (N-1) * 16) / N
node_x(j)    = zone_x + zone_pad_x + j * (node_w + 16)
node_cx(j)   = node_x(j) + node_w/2
```

Para a linha canônica de 3 nós: `node_w = (696 - 32 - 32) / 3 = 210.67`. O exemplo distribuído usa `node_w=160` fixo com posições x personalizadas (`288, 480, 672`), escolhidas para que o `cx` de cada nó se alinhe à coluna por conveniência dos conectores: 368, 560, 752. **Ambos os layouts são válidos**; a fórmula acima é o padrão para novos diagramas. Documente qualquer desvio no SVG renderizado com um comentário.

### 2.3 Posicionamento de barras (largura total da zona)

```
bar_x      = zone_x + zone_pad_x         # 276
bar_w      = zone_w - 2*zone_pad_x       # 664
bar_cx     = zone_cx                     # 608
```

As barras ocupam toda a largura da zona menos 16 px de padding de cada lado. Barras marcadas com `focal: true` usam `bar_h_focal=56` e estilo de destaque (fill `rgba(focal_accent, 0.08)`, stroke `focal_accent`). Barras não focais usam `bar_h_default=44` com estilo atenuado (fill `rgba(35,47,62,0.05)`, stroke `rgba(35,47,62,0.32)`).

### 2.4 Posicionamento de origens / consumidores (colunas laterais)

```
source_y(k)       = col_top + k * 88            # 92, 180, 268, 356, …
source_cy(k)      = source_y(k) + col_node_h/2  # 124, 212, 300, 388, …
consumer_y(k)     = source_y(k)                 # mirrored
consumer_cy(k)    = source_cy(k)
```

Todos os nós das colunas laterais usam tamanho fixo `w=160 h=64`. Mesmo padrão de fill / stroke: fill `rgba(84,91,100,0.06)`, stroke `#7D8998`, stroke-width 1.

---

## 3. Regras de conectores (obrigatórias)

Cinco estilos, vinculados à topologia. Não permita que o usuário substitua o estilo em arestas que tocam componentes focais, que se originam de barras ou que vão do Trino → consumidor — esses são fixados por regra.

| `style` | Traço | Largura | Tracejado | Marcador | Quando é obrigatório |
|---|---|---|---|---|---|
| `primary` | `#EC7211` (focal_accent) | 1.4 | — | `arrow-accent` | Toda aresta cuja extremidade seja um componente `focal: true`. Também toda aresta Trino → consumidor (regra do fluxo de serviço). |
| `secondary` | `#545B64` (muted) | 1.2 | — | `arrow` | Padrão para arestas internas entre componentes da plataforma e para arestas origem → plataforma que não tocam um focal. |
| `federated` | `#0972D3` (link-blue) | 1.0 | `4,3` | `arrow-link` | Consultas de federação (ex.: BD de origem → Trino). |
| `trigger` | `#545B64` (muted) | 1.0 | `4,3` | `arrow` | Toda aresta originada de um componente `kind: bar` (descidas do Airflow). Sem rótulo. |
| `auth` | `#EC7211` | 1.2 | `5,4` | `arrow-accent` | Toda aresta de um nó de rodapé subindo até a borda inferior da zona. **Nunca para um componente específico.** |

**Bloco defs** (obrigatório, cinco marcadores — exatamente):

```svg
<defs>
  <marker id="arrow"        markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#545B64"/></marker>
  <marker id="arrow-accent" markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#EC7211"/></marker>
  <marker id="arrow-link"   markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="#0972D3"/></marker>
  <marker id="arrow-sm"     markerWidth="6" markerHeight="5" refX="5" refY="2.5" orient="auto"><polygon points="0 0, 6 2.5, 0 5" fill="#545B64"/></marker>
  <marker id="arrow-dim"    markerWidth="8" markerHeight="6" refX="7" refY="3"   orient="auto"><polygon points="0 0, 8 3, 0 6" fill="rgba(35,47,62,0.45)"/></marker>
</defs>
```

### 3.1 Lados de saída / entrada (inegociáveis)

| Tipo de aresta | Lado de saída da origem | Lado de entrada do destino |
|---|---|---|
| Origem → componente da plataforma | **direita** da origem | **esquerda** do destino |
| Plataforma → plataforma (mesma linha) | **direita** | **esquerda** |
| Barra → nó da linha (descida vertical) | **base** da barra em `node_cx(target)` | **topo** do destino |
| Plataforma → consumidor | **direita** do componente de plataforma de origem | **esquerda** do consumidor |
| Rodapé → zona | **topo** do rodapé (em `footer_auth_x(k)`) | borda inferior da zona `y = zone_y + zone_h` |
| Rodapé → componente (qualquer componente específico) | **proibido** |

### 3.2 Roteamento

- Cotovelos ortogonais com no máximo duas dobras; Q-bezier `r=8` em cada canto.
- **Escalonamento de fan-out:** quando um nó distribui (fan-out) para N destinos do mesmo lado, escalone o y de saída em ±4 px por índice para que as setas não se sobreponham (ex.: Trino → 4 consumidores sai em y=124, 132, 140, 148). Os segmentos verticais correm no corredor entre a borda da zona e a coluna de consumidores, também escalonados em y.
- **Ordem Z:** todos os conectores são desenhados **antes** de qualquer rect (para que os fills dos nós mascarem as pontas das linhas).
- **Marcadores:** exatamente um `marker-end` por `<line>` / `<path>`. Nunca `marker-start`.
- **Rótulos:** toda aresta `primary`, `secondary`, `federated` e `auth` recebe um rótulo de protocolo (Amazon Ember Mono 8 px, máscara de rect preenchida com paper e folga de 6–10 px acima do traço). Arestas `trigger` ficam sem rótulo.

### 3.3 Tronco rodapé → zona

Com N=1 rodapé: uma única linha vertical em `x = zone_cx` de `footer_y(0)` até `zone_y + zone_h`.

Com N≥2 rodapés: escalone as linhas AUTH para que não se sobreponham aos rodapés empilhados. Para o rodapé de índice `k`:

```
footer_auth_x(k) = zone_cx + (k - (N-1)/2) * 32     # 32-px stride per footer
```

Exemplos:
- N=1 → 560
- N=2 → 544, 576
- N=3 → 528, 560, 592

Cada linha AUTH vai de `(footer_auth_x(k), footer_y(k))` até `(footer_auth_x(k), zone_y + zone_h)`. Os rótulos AUTH ficam logo acima da ponta da seta na borda inferior da zona.

### 3.4 Cruzamentos

Evite. Redirecione pelas posições x do corredor antes de aceitar um cruzamento. Se for inevitável, o caminho desenhado em segundo lugar recebe um salto em arco de 6 px (hop) sobre o primeiro.

---

## 4. Substituição de cor de componente (espelha `type-high-level.md` §3.4)

Qualquer origem, consumidor, componente da plataforma (nó ou barra) ou nó de rodapé aceita um `color: "#hex"` opcional. Espelha o high-level para que a regra seja lida de forma idêntica entre os tipos.

**Onde a cor é aplicada** (`C = color`):

| Elemento | Claro | Escuro |
|---|---|---|
| Fill do contêiner (corpo do `rect`) | `rgba(C, 0.06)` | `rgba(C_light, 0.10)` |
| Stroke do contêiner | `rgba(C, 0.35)` (`stroke-width=1` para nós, `0.8` para barras) | `rgba(C_light, 0.45)` |
| Stroke do badge de papel | `rgba(C, 0.40)` | `rgba(C_light, 0.55)` |
| Texto do badge de papel | `rgba(C, 0.85)` | `rgba(C_light, 1.0)` |
| Stroke / fill do ícone | `C` | `C_light` |
| Texto do nome | `C` | `C_light` |
| Texto do subtítulo | **inalterado** (muted) | **inalterado** (muted) |
| Conectores que tocam este componente | **inalterados** — definidos pela topologia | **inalterados** |

`C_light` = o mesmo hex clareado em ~15% para contraste no modo escuro (ex.: `#b85450` → `#d97a78`).

**Regras:**

- **Nunca em componentes focais.** `focal_accent` sempre prevalece — um `color` em um componente focal é ignorado.
- **Nunca em conectores.** Se você quer uma aresta colorida, escolha outro `style` da §3, não uma substituição de cor.
- **Limite de 2 componentes com cor personalizada** por diagrama (além do par focal).

**Paleta semântica** (use estas, a menos que a marca exija outra coisa):
- `#b85450` vermelho-ferrugem — Segurança / Identidade (AD, Keycloak, Vault)
- `#5a7d9a` azul-ardósia — Observabilidade (Prometheus, Datadog, OpenTelemetry)
- `#7a8c47` verde-oliva — Governança / Linhagem (OpenMetadata, DataHub)
- `#8c6d3f` marrom-quente — Backup / DR (Velero, Restic)

---

## 5. Regra focal

**Exatamente dois componentes focais.** Padrão: o hub de armazenamento (MinIO / S3 / similar) e o motor de federação (Trino / Dremio / similar). Essas duas superfícies distinguem uma "plataforma" de uma pilha de ferramentas. Todo o resto (NiFi, Jupyter, Airflow, AD, todas as origens, todos os consumidores) permanece em ink / muted.

- Marque com `focal: true` na entrada do componente.
- Um `kind: bar` focal usa `bar_h_focal=56` (mais alto) e estilo de destaque.
- Um nó focal de `kind: row` mantém `row_h=72`, mas usa estilo de destaque.
- As arestas **Trino → todos os consumidores** são sempre `primary` (accent), independentemente da flag focal de cada consumidor — esta é a regra do fluxo de serviço.
- Se menos de 2 ou mais de 2 componentes estiverem marcados com `focal: true`, interrompa e pergunte ao usuário.

---

## 6. Modo escuro

| Token | Claro | Escuro |
|---|---|---|
| Papel da página (paper) | `#ffffff` | `#161E2D` |
| Ink | `#232F3E` | `#ffffff` |
| Muted | `#545B64` | `#B6BEC9` |
| Accent | `#EC7211` | `#FF9900` |
| Link (federado) | `#0972D3` | `#539FE5` |
| Fill da coluna lateral | `rgba(84,91,100,0.06)` | `rgba(255,255,255,0.06)` |
| Stroke da coluna lateral | `#7D8998` | `rgba(255,255,255,0.30)` |
| Fill da zona | `rgba(35,47,62,0.025)` | `rgba(255,255,255,0.04)` |
| Stroke da zona | `rgba(35,47,62,0.32)` | `rgba(255,255,255,0.30)` |
| Fill de barra não focal | `rgba(35,47,62,0.05)` | `rgba(255,255,255,0.06)` |
| Fill focal | `rgba(255,153,0,0.08)` | `rgba(255,153,0,0.12)` |
| Stroke focal | `#EC7211` | `#FF9900` |
| Cores personalizadas de componente | `C` | `C_light` (clarear ~15%) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. `viewBox = "0 0 1200 {viewBox_h}"` onde `viewBox_h = max(600, footer_bottom + 84)`.
2. Zona da plataforma em `x=260 y=72 w=696 h=col_h`. Rótulo da zona mascarado com paper sobre a borda superior em `y=zone_y+3`.
3. Coluna esquerda em `x=40..200`, coluna direita em `x=1000..1160` — ambas com 160 de largura.
4. Topo das linhas de origens / consumidores em `y=92`, passo de 88 px.
5. As entradas de `platform.rows` são empilhadas dentro da zona pelo algoritmo de cursor da §2.1; a extensão total em y ≤ `zone_h`.
6. Dentro de cada `kind: row`, os centros x dos nós são espaçados uniformemente pela largura da zona (§2.2).
7. **Exatamente 2** componentes focais (`focal: true`).
8. Toda aresta originada de um componente `kind: bar` usa `style: trigger` (tracejada, sem rótulo).
9. Toda aresta Trino → consumidor usa `style: primary` (a regra do fluxo de serviço).
10. Os nós de rodapé se conectam apenas à borda inferior da zona via estilo `auth`. **Nenhuma** aresta de um rodapé para um componente específico.
11. Cores personalizadas de componente ≤ 2 (além do par focal). Conectores nunca são recoloridos pelo `color` do componente.
12. Todos os conectores emitidos antes de qualquer rect de nó (regra de ordem Z).

---

## 8. Origens e consumidores — biblioteca de ícones

Defina cada ícone como `<g id="ico-…">` em `<defs>`, desenhado em translate(cx, cy) com `stroke="currentColor"` para que herde a cor do texto ao redor. Ícones comuns:

- `ico-db` (cilindro) — origens relacionais
- `ico-sftp` (pasta com seta para baixo) — depósitos de arquivos
- `ico-mail` (envelope) — coletas de e-mail
- `ico-mainframe` (servidor com grades de ventilação) — sistemas legados
- `ico-monitor` — ferramentas analíticas de desktop
- `ico-chart` (barras) — ferramentas de BI / relatórios
- `ico-globe` — sites públicos
- `ico-api` (chaves `{}`) — gateways e clientes de terceiros
- `ico-key` — identidade / IDP
- `ico-monitoring` (gráfico de linha) — stack de observabilidade

Se precisar de mais ícones, consulte `assets/icons.html` e defina os blocos `<symbol>` correspondentes.

---

## 9. Identidade e serviços comuns → conecte à camada, não aos componentes

**Active Directory** (ou Keycloak, IAM, OPA, qualquer repositório transversal de identidade / políticas / segredos) autentica *todos* os componentes da plataforma. Conectá-lo a uma ferramenta específica subestimaria o escopo de confiança. Em vez disso, conecte-o com uma única seta até a borda inferior da zona da plataforma, com o rótulo `AUTH` (§3.3).

A mesma regra vale para qualquer outro serviço de abrangência de camada: logging centralizado, cofre de segredos, stack de observabilidade, destino de auditoria, raiz mTLS. Cada um vai na lista `footer`, cada um recebe sua própria linha, cada um recebe sua própria linha AUTH até a borda inferior da zona (escalonada por índice conforme a §3.3). A leitura visual é "a camada da plataforma delega a todos esses", que é a verdade arquitetural.

---

## 10. Orçamento — este tipo excede o padrão

Este é o único tipo em que o orçamento (limite) padrão de 9 nós / 12 setas é excedido intencionalmente. Uma integração de plataforma realista mostra:

- 4–6 nós de origem
- 5 componentes de plataforma
- 4–6 nós de consumo
- 1–3 nós de rodapé (identidade, observabilidade, backup, …)

Isso dá **14–20 nós**. A complexidade é o objetivo — o diagrama faz uma afirmação sobre o *número de superfícies de integração distintas*. Comprimi-las anula a afirmação.

Quando isso ficar difícil de manejar:
- Combine linhas de origem claramente idênticas (ex.: quatro bancos MariaDB → um nó `Databases` com sub-rótulo `4 × MariaDB`)
- Divida em dois diagramas (um por plano de integração: dados vs. identidade vs. observabilidade)

---

## 11. Antipadrões

- **Origens ou consumidores como um único nó colapsado** quando existem ≥3 itens distintos — anula todo o propósito deste tipo. Use Architecture ou High-level se quiser colapsar.
- **Uma única seta de barramento de "origens" para "a plataforma"** — cada conexão é rotulada com seu protocolo; é assim que equipes de integração leem o diagrama.
- **Codificação de cor por ferramenta** (NiFi em verde-azulado, MinIO em magenta, Jupyter em amarelo) dentro da zona — colapsa a hierarquia; apenas os dois destaques focais ganham coral, mais até 2 cores personalizadas em componentes transversais (limite da §4).
- **Mais de 2 componentes focais** — o focal existe para distinguir "plataforma" de "pilha de ferramentas"; >2 apaga o sinal (mesma regra de SKILL.md §1).
- **Substituição de `color` em um componente focal** — ignorada. O focal_accent sempre prevalece.
- **Rodapé conectado a uma ferramenta específica** (ex.: AD → apenas Airflow) — errado, a menos que esse serviço realmente proteja só uma ferramenta. O padrão é a conexão com a camada inteira.
- **Rodapé ou identidade dentro da zona** — a identidade controla o acesso à camada a partir de fora. Desenhá-la dentro representa incorretamente o modelo de confiança.
- **Chevrons de fase no topo** — esses pertencem ao `high-level`.
- **Conectores com cor personalizada** — conectores são definidos pela topologia. O estilo escolhe a cor; o `color` de um componente nunca se propaga para suas arestas.

---

## 12. Exemplos

- `assets/example-dp-integration.html` — mínimo claro (1 rodapé = AD). Padrão da galeria.
- `assets/example-dp-integration-dark.html` — o mesmo, skin (tema) escuro.
- `assets/example-dp-integration-full.html` — o mesmo, moldura de cartão editorial.
- `assets/example-dp-integration-extended.html` — exercita a substituição de cor da §4 + múltiplos rodapés: AD em vermelho-ferrugem, Observabilidade (Prometheus/Grafana/Loki) em azul-ardósia. Altura do canvas aumentada para acomodar 2 linhas de rodapé.
- `assets/example-dp-integration-extended-dark.html` — padrão estendido, skin (tema) escuro.
- `assets/example-dp-integration-extended-full.html` — padrão estendido, moldura de cartão editorial.
