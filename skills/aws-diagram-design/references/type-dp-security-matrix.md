# Matriz de segurança DP (DP security matrix)

**Ideal para:** documentar permissões de acesso por papel / por componente de uma plataforma de dados — uma grade em que cada linha é um componente da plataforma (Keycloak, bucket MinIO, catálogo Trino, JupyterHub, NiFi, …) e cada coluna é um papel / grupo do AD (Data Administrators, Data Engineers, Data Scientists, Data Consumers, …). Cada célula de interseção contém um valor de permissão (Admin / Full / R/W / Read / SELECT / Login / No access) com uma categoria visual que corresponde ao nível de permissão. Uma célula pode ser marcada como focal para sinalizar uma regra de acesso crítica (ex.: "Data Consumers podem SOMENTE fazer `SELECT` no catálogo agregado — único acesso de consumidor").

Use quando os stakeholders precisarem auditar *quem pode fazer o quê* em toda a plataforma. Prefira **DP integration** quando a pergunta for *quem pode falar com o quê* (topologia/protocolo) em vez de *quem pode escrever/ler o quê* (permissões).

Este tipo é **paramétrico** — o schema de entradas em §1 determina cada coordenada por meio das fórmulas em §2. O formato das regras espelha `type-medallion.md` / `type-process.md` / `type-data-flow.md`, para que a regra focal, a sobrescrita de cor, o modo escuro e o checklist de reprodutibilidade sejam lidos de forma idêntica entre os tipos.

---

## 1. Entradas — o contrato de parâmetros

```yaml
title:    "Platform Access Matrix"
subtitle: "Four canonical groups × platform components"

roles:                                  # 2..6 columns, ordered left → right
  - { name: "Data Administrators", code: "DL-DataAdmins"      }
  - { name: "Data Engineers",      code: "DL-DataEngineers"   }
  - { name: "Data Scientists",     code: "DL-DataScientists"  }
  - { name: "Data Consumers",      code: "DL-DataConsumers"   }

components:                             # 2..14 rows, ordered top → bottom
  - { name: "Keycloak",                          hint: "SSO" }   # `hint` = right-aligned aside in label cell
  - { name: "MinIO · raw bucket" }
  - { name: "MinIO · anon · staging · agg" }
  - { name: "Trino · raw catalog" }
  - { name: "Trino · anon-staging" }
  - { name: "Trino · aggregated" }
  - { name: "JupyterHub" }
  - { name: "NiFi" }

cells:                                  # explicit (row, col) entries; omitted → defaults to "none"
  # value = displayed text (free-form)
  # level = visual category: full | rw | read | none  (closed vocabulary, drives styling)
  # focal: true (max 1)            — overrides level to focal styling
  # sub: "second-line text"        — used inside focal cell
  # color: "#hex"                  — optional per-cell color override (§4)
  - { row: 0, col: 0, value: "Admin", level: "full" }
  - { row: 0, col: 1, value: "Login", level: "read" }
  - { row: 0, col: 2, value: "Login", level: "read" }
  - { row: 0, col: 3, value: "Login", level: "read" }

  - { row: 1, col: 0, value: "Full",       level: "full" }
  - { row: 1, col: 1, value: "R/W",        level: "rw"   }
  - { row: 1, col: 2, value: "No access",  level: "none" }
  - { row: 1, col: 3, value: "No access",  level: "none" }

  # ... rows 2..4 follow the same pattern ...

  - { row: 5, col: 0, value: "Full",       level: "full" }
  - { row: 5, col: 1, value: "R/W",        level: "rw"   }
  - { row: 5, col: 2, value: "SELECT",     level: "read" }
  - { row: 5, col: 3, value: "SELECT only", sub: "sole consumer access", focal: true }

  # ... rows 6..7 ...

none_label: "No access"                 # default text rendered when a cell is omitted
dark: false
```

**Semântica dos campos reservados:**
- `roles[j].name` — rótulo principal do papel (papel `node-name` em 11px, texto branco sobre o banner `ink`)
- `roles[j].code` — identificador secundário do grupo do AD (papel `sublabel`, texto branco com opacidade 0.85)
- `components[i].hint` — texto `sublabel` opcional, alinhado à direita na célula de rótulo (ex.: `"SSO"`, `"S3 API"`)
- `cells[k].level` — vocabulário fechado `full | rw | read | none`. Determina preenchimento/traço/cor do texto conforme §2.4.
- `cells[k].value` — texto de exibição livre. Rótulos específicos do domínio (`"R/W"`, `"SELECT"`, `"Login"`) funcionam sem inventar novos níveis.
- `cells[k].focal: true` — exatamente **uma** célula pode declarar isto. Sobrescreve `level` com o estilo focal.
- `cells[k].sub` — texto opcional de 2ª linha (usado com focal). Renderizado no papel `sublabel` em 8px abaixo do valor principal.
- `cells[k].color: "#hex"` — sobrescrita de cor opcional por célula (§4).

---

## 2. Fórmulas de layout — geometria determinística

```
# Constants
left_pad         = 12
right_pad        = 48
comp_col_w       = 208
comp_role_gap    = 12
role_col_w       = 148
role_col_gap     = 16
header_h         = 52
row_h            = 36
row_stride       = 40

# Counts
n_roles          = len(roles)        # 2..6
n_components     = len(components)   # 2..14

# Canvas
viewBox_w        = left_pad + comp_col_w + comp_role_gap
                   + n_roles * role_col_w + (n_roles - 1) * role_col_gap
                   + right_pad
                   # 4 roles → 12 + 208 + 12 + 592 + 48 + 48 = 920

header_y         = 72
row_y(k)         = 140 + k * row_stride                  # 140, 180, 220, ...
rows_bottom      = row_y(n_components - 1) + row_h       # 8 rows → 456
legend_y_top     = rows_bottom + 20                      # 476 for 8-row canonical
viewBox_h        = legend_y_top + 44                     # 520 for 8-row canonical

# Column positions
comp_col_x       = left_pad                                                       # 12
role_col_x(j)    = left_pad + comp_col_w + comp_role_gap
                   + j * (role_col_w + role_col_gap)
                                                                                  # 232, 396, 560, 724
role_col_cx(j)   = role_col_x(j) + role_col_w / 2                                 # 306, 470, 634, 798
```

### 2.1 Fundo

Preenchimento sólido `paper` em todo o *viewBox*. Sem padrão de pontos.

### 2.2 Linha de cabeçalho (`y = 72, h = 52`)

**Célula de cabeçalho da coluna de componentes:**
- Rect: `(comp_col_x, header_y, comp_col_w, header_h)`, preenchimento branco, traço `ink @ 0.12` 0.8, `rx=6`
- Rótulo de duas linhas centralizado em `(comp_col_x + comp_col_w/2, header_y+24)` e `(…, header_y+40)`:
  - Linha 1: `"Component"` — papel `node-name` em 11px, `ink`
  - Linha 2: `"vs. AD group"` — papel `sublabel`, `muted`

**Banners de papel (um por papel):**
- Rect: `(role_col_x(j), header_y, role_col_w, header_h)`, preenchimento `ink`, `rx=6`
- Rótulo de duas linhas centralizado:
  - Linha 1 em `y=92`: `roles[j].name` — papel `node-name` em 11px, branco
  - Linha 2 em `y=108`: `roles[j].code` — papel `sublabel`, branco com opacidade 0.85

### 2.3 Linha de dados (`y = row_y(k), h = 36`)

**Célula de rótulo do componente:**
- Rect: `(comp_col_x, row_y(k), comp_col_w, row_h)`, preenchimento branco, traço `ink @ 0.12` 0.8, `rx=4`
- Nome em `(comp_col_x + 12, row_y(k) + 22)`: papel `node-name` em 11px, `ink`, alinhado à esquerda
- Hint (se presente) em `(comp_col_x + comp_col_w − 12, row_y(k) + 22)`: papel `sublabel`, `muted`, alinhado à direita

**Células de valor (uma por papel × componente):**
- Rect: `(role_col_x(j), row_y(k), role_col_w, row_h)`, `rx=4`, traço `ink @ 0.12` 0.6
- O preenchimento e a cor do texto dependem de `level` (ou da flag focal) — veja §2.4
- Texto do valor centralizado em `(role_col_cx(j), row_y(k) + 22)`: papel `node-name` em 10px
- A célula focal usa o texto principal ligeiramente elevado em `y=18` e uma sublinha em `y=30` (papel `sublabel` em 8px)

### 2.4 Tabela de estilos das células

| `level` | Preenchimento | Traço | Cor do texto | Peso do texto |
|---|---|---|---|---|
| `full` | `ink @ 0.08` | `ink @ 0.12` | `ink` | 600 |
| `rw` | `#FFFFFF` | `ink @ 0.12` | `ink` | 400 |
| `read` | `muted @ 0.08` | `ink @ 0.12` | `muted` | 400 |
| `none` | `paper` | `ink @ 0.12` | `soft` | 400 |
| **focal** | `accent @ 0.07` | `accent` (1.4) | `accent` | 600 |

A célula focal pode ter uma 2ª linha (`sub:`) renderizada em `accent` com o papel `sublabel` em 8px e opacidade 0.85.

### 2.5 Legenda (`y_top = legend_y_top, h ≈ 30`)

Separador de linha fina (hairline) em `legend_y_top`. Abaixo do separador, uma linha de amostras de estilo com seus rótulos — somente os estilos efetivamente usados no diagrama aparecem na legenda.

- Eyebrow "LEGEND" em `(left_pad, legend_y_top + 20)`: papel `eyebrow`, `muted`, letter-spacing 0.14em
- Cada estilo: rect de amostra (14×12 `rx=2`) seguido de um rótulo no papel `sublabel`
- As posições x dos itens são tabuladas da esquerda para a direita com passo de ~120 px; a legenda quebra para uma segunda linha visual somente se `n_roles ≥ 6` (caso contrário, cabe em uma linha)

---

## 3. Células, não conectores

Um diagrama de matriz **não tem conectores** — não há setas entre células, nem linhas de fluxo. A informação do diagrama está inteiramente no conteúdo + estilo das células. O único elemento "parecido com conector" é a borda `accent` da célula focal, que visualmente "destaca" uma interseção específica.

As células **não emitem arestas**. Não adicione setas apontando para células ou entre células — elas pertencem a outro tipo de diagrama.

---

## 4. Sobrescritas de cor

Três eixos de sobrescrita independentes — por célula, por componente (linha), por papel (coluna). Todos opcionais, todos usam o mesmo campo `color: "#hex"`, todos tiram da mesma paleta recomendada. Espelha §3.4 de `type-high-level.md`, §4 de `type-process.md`, §4 de `type-medallion.md`.

### 4.1 `color` por célula

Tinge uma célula de interseção específica. Aplicado a:

| Elemento | Claro | Escuro |
|---|---|---|
| Preenchimento da célula | `rgba(C, 0.08)` | `rgba(C_light, 0.12)` |
| Traço da célula | `rgba(C, 0.45)` largura 1.0 | `rgba(C_light, 0.55)` largura 1.0 |
| Texto do valor | `C` | `C_light` |
| Texto sub (se presente) | `rgba(C, 0.85)` | `rgba(C_light, 0.95)` |

`C_light` = o mesmo hex clareado ~15% para contraste no modo escuro.

### 4.2 `color` por componente  (`components[i].color`)

Tinge **somente a célula de rótulo** da linha (coluna da esquerda). As células de dados dessa linha mantêm o estilo de `level` por célula — a cor da linha sinaliza *o que* é este componente, não *quais permissões existem nele*.

| Elemento | Claro | Escuro |
|---|---|---|
| Preenchimento da célula de rótulo | `rgba(C, 0.06)` | `rgba(C_light, 0.10)` |
| Traço da célula de rótulo | `rgba(C, 0.45)` largura 0.8 | `rgba(C_light, 0.55)` largura 0.8 |
| Nome do componente | `C` | `C_light` |
| Texto do hint | inalterado (`muted`) | inalterado (`muted`) |

### 4.3 `color` por papel  (`roles[j].color`)

Tinge **somente o banner da coluna** (linha superior). As células abaixo mantêm o estilo de `level`. Substitui o preenchimento azul-marinho padrão do banner pelo hex escolhido.

| Elemento | Claro e Escuro (o banner é igual em ambos os modos) |
|---|---|
| Preenchimento do banner | `C` |
| Texto do nome + código do papel | `#FFFFFF` se `C` for escuro (luminância ≤ 0.5), senão `ink` |

Se você escolher um hex de luminância média (ex.: amarelo `#c9a23a`), o texto muda automaticamente para `ink` para dar contraste. Combine com `roles[j].text_color: "#hex"` para sobrescrever essa escolha automática.

### 4.4 Regras

- **A célula focal vence.** Uma célula focal ignora sobrescritas de `color` — sempre `accent`.
- **`color` por célula sobrescreve o estilo de `level`** para aquela célula.
- **Sobrescritas por componente / por papel têm escopo:** componente → somente o rótulo da linha; papel → somente o banner. Elas **não** se espalham pelo corpo da matriz. Para sinalizar uma interseção específica, use por célula.
- **Limite:** mantenha o total de entidades com cor personalizada ≤ 5 por diagrama (somando células + componentes + papéis). Acima de 5, a matriz vira ruído colorido — divida em vários diagramas ou repense qual cor carrega qual preocupação.

### 4.5 Paleta recomendada (a mesma dos outros tipos paramétricos)

- `#b85450` vermelho-ferrugem — Elevação de segurança / break-glass / sinalizado por SoX
- `#5a7d9a` azul-ardósia — Gate de qualidade / monitoramento / observabilidade
- `#7a8c47` verde-oliva — Aprovado / liberado pela governança / pronto para publicação
- `#c9a23a` amarelo quente — Trabalho / sandbox / zona de data scientists
- `#8c6d3f` marrom quente — Arquivo / frio / DR

---

## 5. Regra focal

Exatamente **uma** célula focal por diagrama (ou zero). A célula focal:
- Usa o estilo focal (preenchimento `accent` + traço `accent` 1.4 + texto `accent` em negrito)
- Pode ter conteúdo de 2 linhas: `value` principal em `y = row_y(k) + 18`, `sub` em `y = row_y(k) + 30`
- Destaca a afirmação central de segurança do diagrama — a *única* regra de acesso que distingue a postura desta plataforma de uma tabela genérica de permissões

Se forem declaradas zero ou >1 células com `focal: true`, pare e pergunte ao usuário.

---

## 6. Modo escuro

| Token | Claro | Escuro |
|---|---|---|
| Paper | `paper` | `ink` |
| Ink | `ink` | `paper` |
| Muted | `muted` | `soft` |
| Soft (texto sem acesso) | `soft` | `muted` |
| Accent | `accent` | `accent` |
| Preenchimento do banner de papel | `ink` | `ink` |
| Traço de cabeçalho / linha | `ink @ 0.12` | `paper @ 0.18` |
| Preenchimento Full / Admin | `ink @ 0.08` | `paper @ 0.10` |
| Preenchimento R/W | `#FFFFFF` | `paper @ 0.06` |
| Preenchimento Read | `muted @ 0.08` | `soft @ 0.12` |
| Preenchimento No-access | `paper` | `paper @ 0.02` |
| Preenchimento focal | `accent @ 0.07` | `accent @ 0.12` |
| Traço focal | `accent` | `accent` |
| Cores personalizadas de componentes | `C` | `C_light` (clarear ~15%) |

---

## 7. Checklist de reprodutibilidade (*taste gate*)

Antes de emitir o SVG, verifique **cada** item:

1. `viewBox = "0 0 {viewBox_w} {viewBox_h}"` derivado via §2 (4 papéis × 8 componentes → 920 × 520).
2. Linha de cabeçalho em `y=72 h=52`. Célula de cabeçalho de componentes com preenchimento branco e rótulo de duas linhas `Component / vs. AD group`. Banners de papel preenchidos com `ink`, com nome + código do grupo do AD em branco.
3. As linhas de dados começam em `y=140`, passo 40, altura 36. `rows_bottom = 140 + (n_components−1)·40 + 36`.
4. Célula de rótulo do componente `rx=4`, nome alinhado à esquerda em `x=24`, `hint` opcional alinhado à direita em `x = comp_col_x + comp_col_w − 12`.
5. Toda célula de valor com `rx=4`, traço `ink @ 0.12` 0.6, preenchimento + texto conforme §2.4 para seu `level`.
6. Exatamente **uma** célula focal (ou zero). Traço da célula focal `accent` com largura 1.4. Valor principal em `y = row_y(k) + 18`; `sub` (se presente) em `y = row_y(k) + 30`.
7. Células omitidas de `cells:` são renderizadas como `level: "none"` com o texto de `none_label` (padrão `"No access"`).
8. Células com cor personalizada ≤ 2 (além da célula focal).
9. Nenhum elemento conector em nenhum lugar do SVG.
10. Faixa de legenda em `legend_y_top`, uma amostra por `level` efetivamente usado, separador de linha fina (hairline) acima.
11. `viewBox_h` cresce com `n_components`; `viewBox_w` cresce com `n_roles`.

---

## 8. Antipadrões

- **Mais de uma célula focal** — o focal existe para marcar *a* regra de acesso crítica; >1 apaga o sinal.
- **Conectores em qualquer lugar** — a matriz é guiada por valores; setas pertencem a diagramas de DP integration / processo.
- **Valores de `level` livres** — o vocabulário fechado é `full | rw | read | none`. Use `value` para texto exibido livre + sobrescrita de `color` para tingimento arbitrário.
- **Tingimento por linha ou por coluna** — aplique `color` somente por célula. Destacar a linha inteira ou a coluna inteira tende a enfatizar demais e reduz a matriz a uma lista.
- **Usar `none_label` como placeholder para "TBD"** — `none` significa *sem acesso*. Se a permissão for desconhecida, deixe a célula vazia nas entradas, mas documente isso em outro lugar; não renderize um estado ambíguo.
- **Mais de 6 papéis** — divida em duas matrizes (ex.: papéis humanos vs contas de serviço) antes de passar de 6 colunas.
- **Mais de 14 componentes** — divida por domínio (armazenamento / computação / observabilidade / governança) antes de passar de 14 linhas.
- **Usar a matriz para documentar *como* as permissões são concedidas** — isso pertence a um diagrama de processo ou de sequência. A matriz mostra *o que* cada papel pode fazer, não o fluxo de concessão.

---

## 9. Exemplos

- `assets/example-dp-security-matrix.html` — mínimo claro (canônico NatStat: 4 papéis × 8 componentes, focal em Data Consumers × Trino aggregated). Padrão da galeria.
- `assets/example-dp-security-matrix-dark.html` — o mesmo, skin (tema) escuro.
- `assets/example-dp-security-matrix-full.html` — o mesmo, moldura de cartão editorial com subtítulo + cartões de resumo.

---

## 10. YAML completo — entradas completas para `example-dp-security-matrix.html`

As entradas completas que correspondem ao exemplo canônico distribuído. Cada coordenada naquele SVG pode ser derivada de §2 aplicado a estas entradas.

```yaml
title:    "Platform Access Matrix"
subtitle: "Four canonical groups × platform components"

roles:
  - { name: "Data Administrators", code: "DL-DataAdmins"      }
  - { name: "Data Engineers",      code: "DL-DataEngineers"   }
  - { name: "Data Scientists",     code: "DL-DataScientists"  }
  - { name: "Data Consumers",      code: "DL-DataConsumers"   }

components:
  - { name: "Keycloak",                          hint: "SSO" }
  - { name: "MinIO · raw bucket" }
  - { name: "MinIO · anon · staging · agg" }
  - { name: "Trino · raw catalog" }
  - { name: "Trino · anon-staging" }
  - { name: "Trino · aggregated" }
  - { name: "JupyterHub" }
  - { name: "NiFi" }

cells:
  # Row 0 — Keycloak
  - { row: 0, col: 0, value: "Admin", level: "full" }
  - { row: 0, col: 1, value: "Login", level: "read" }
  - { row: 0, col: 2, value: "Login", level: "read" }
  - { row: 0, col: 3, value: "Login", level: "read" }
  # Row 1 — MinIO raw
  - { row: 1, col: 0, value: "Full", level: "full" }
  - { row: 1, col: 1, value: "R/W",  level: "rw"   }
  - { row: 1, col: 2, value: "No access", level: "none" }
  - { row: 1, col: 3, value: "No access", level: "none" }
  # Row 2 — MinIO anon/staging/agg
  - { row: 2, col: 0, value: "Full", level: "full" }
  - { row: 2, col: 1, value: "R/W",  level: "rw"   }
  - { row: 2, col: 2, value: "Read", level: "read" }
  - { row: 2, col: 3, value: "No access", level: "none" }
  # Row 3 — Trino raw catalog
  - { row: 3, col: 0, value: "Full", level: "full" }
  - { row: 3, col: 1, value: "R/W",  level: "rw"   }
  - { row: 3, col: 2, value: "No access", level: "none" }
  - { row: 3, col: 3, value: "No access", level: "none" }
  # Row 4 — Trino anon-staging
  - { row: 4, col: 0, value: "Full",   level: "full" }
  - { row: 4, col: 1, value: "R/W",    level: "rw"   }
  - { row: 4, col: 2, value: "SELECT", level: "read" }
  - { row: 4, col: 3, value: "No access", level: "none" }
  # Row 5 — Trino aggregated (focal cell at col 3)
  - { row: 5, col: 0, value: "Full",        level: "full" }
  - { row: 5, col: 1, value: "R/W",         level: "rw"   }
  - { row: 5, col: 2, value: "SELECT",      level: "read" }
  - { row: 5, col: 3, value: "SELECT only", sub: "sole consumer access", focal: true }
  # Row 6 — JupyterHub
  - { row: 6, col: 0, value: "Admin", level: "full" }
  - { row: 6, col: 1, value: "R/W",   level: "rw"   }
  - { row: 6, col: 2, value: "R/W",   level: "rw"   }
  - { row: 6, col: 3, value: "No access", level: "none" }
  # Row 7 — NiFi
  - { row: 7, col: 0, value: "Admin", level: "full" }
  - { row: 7, col: 1, value: "R/W",   level: "rw"   }
  - { row: 7, col: 2, value: "Read",  level: "read" }
  - { row: 7, col: 3, value: "No access", level: "none" }

dark: false
```

### 10.1 O que este YAML comprova

Execute §2 com estas entradas:
- `n_roles = 4`, `n_components = 8`, sem sobrescritas de cor, uma célula focal.
- `viewBox_w = 12 + 208 + 12 + 4·148 + 3·16 + 48 = 920` ✓
- `row_y(k)` produz `140, 180, 220, 260, 300, 340, 380, 420` ✓
- `rows_bottom = 420 + 36 = 456`; `legend_y_top = 476`; `viewBox_h = 520` ✓
- `role_col_x(j) = [232, 396, 560, 724]` ✓
- Célula focal em `(row=5, col=3)` → rect `(724, 340, 148, 36)` com traço `accent` 1.4 ✓

Uma geração nova a partir deste YAML produz um diagrama visualmente indistinguível do `example-dp-security-matrix.html` distribuído.
