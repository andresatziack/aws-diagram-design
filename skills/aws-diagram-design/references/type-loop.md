# Loop (ciclo de reforço)

**Ideal para:** ciclos de reforço, flywheels, loops de feedback e loops operacionais — qualquer coisa em que a última etapa alimenta a primeira e um hub compartilhado acumula estado. Use Loop quando o leitor precisar ver os dois movimentos ao mesmo tempo: o trabalho avança no sentido horário ao redor do anel, enquanto cada passagem grava estado durável de volta em um centro comum.

Prefira **Flowchart** quando o caminho termina, ramifica em direção a um resultado ou nunca retorna de fato à sua primeira etapa. Prefira **Cycle** quando o centro não acumula estado compartilhado. Os raios tracejados de gravação de volta (write-back) são o sinal definidor aqui: remova-os e a figura passa a ser apenas um processo circular.

Este tipo é **paramétrico**. As entradas em §1 determinam a quantidade de estações, os ângulos, as interseções com as bordas, os caminhos dos conectores e os limites do *viewBox*. Entradas idênticas devem produzir geometria idêntica.

---

## 1. Entradas — o contrato de parâmetros

```yaml
title: "The self-improving loop"
subtitle: "Every pass improves the shared operating record"

hub:                                  # exactly one
  name: "Shared memory"
  sublabel: "one record, every loop"

stations:                             # 5..8, clockwise from top
  - { name: "Capture",  sublabel: "signals in",       spoke_label: "SIGNALS" }
  - { name: "Research", sublabel: "evidence pulled" }
  - { name: "Decide",   sublabel: "human approves",   focal: true }
  - { name: "Act",      sublabel: "work ships",        spoke_label: "OUTCOMES" }
  - { name: "Measure",  sublabel: "outcomes logged" }
  - { name: "Learn",    sublabel: "playbook updated" }

station_w: 160
station_h: 64
hub_w: 200
hub_h: 104
radius: 240
margin: 64
dark: false
```

**Orçamento (limite rígido):** **5–8 estações mais exatamente um hub.** Acima de 8 estações, divida o assunto em um Loop de visão geral e um ou mais diagramas de detalhe. Exatamente um hub — um loop com dois hubs são dois diagramas. No máximo uma estação pode definir `focal: true`; zero é permitido quando nenhum ponto de decisão editorial merece ênfase.

A ordem das estações é semântica. `stations[0]` é a estação do topo, e as entradas seguem no sentido horário. A última estação sempre se conecta de volta à estação 0; se esse retorno for falso, use um Flowchart.

---

## 2. Matemática do layout — geometria determinística

Use coordenadas SVG, em que y positivo aponta para baixo. Seja o centro do hub `C = (cx, cy)`, a quantidade de estações `N`, o raio do anel de estações `R`, a meia-dimensão da estação `a = station_w/2`, `b = station_h/2`, e a meia-dimensão do hub `A = hub_w/2`, `B = hub_h/2`.

### 2.1 Centros das estações

Para a estação `k` (indexada a partir de zero):

```text
theta_k = -90deg + k * (360deg / N)
u_k     = (cos(theta_k), sin(theta_k))
P_k     = C + R * u_k

station_center_x(k) = cx + R * cos(theta_k)
station_center_y(k) = cy + R * sin(theta_k)
station_x(k)        = station_center_x(k) - station_w/2
station_y(k)        = station_center_y(k) - station_h/2
```

Assim, a estação 1 fica no topo, e aumentar `k` move no sentido horário. Arredonde os retângulos das estações para o ponto mais próximo da grade de 4px depois de calcular a geometria ideal; preserve a simetria ao arredondar estações em pares. Mantenha os pontos de interseção entre o anel e o círculo com três casas decimais, para que todo arco permaneça no mesmo círculo.

### 2.2 Extremidades do fluxo sólido do anel

Os conectores do anel vão da estação `k` para a estação `j = (k + 1) mod N` como arcos SVG circulares sobre o próprio círculo das estações. Todo segmento usa o mesmo centro `C`, raio `R` e varredura no sentido horário. As caixas das estações interrompem o círculo; os conectores começam na saída horária do círculo a partir da caixa de origem e terminam logo antes da entrada anti-horária na caixa de destino, de modo que a ponta do marcador caia sobre o traço do destino.

Encontre as interseções círculo/retângulo com as quatro bordas da estação `k`. Para a borda vertical `x = x_e`:

```text
y = cy +/- sqrt(R^2 - (x_e - cx)^2)
```

Mantenha apenas os candidatos cujo `y` esteja dentro da borda. Para a borda horizontal `y = y_e`:

```text
x = cx +/- sqrt(R^2 - (y_e - cy)^2)
```

Mantenha apenas os candidatos cujo `x` esteja dentro da borda. Os dois pontos restantes são classificados por seus ângulos polares normalizados em torno de `C`:

```text
q_entry(k) = circle/box intersection immediately before theta_k clockwise
q_exit(k)  = circle/box intersection immediately after  theta_k clockwise
```

Compense a ponta do marcador antes de emitir a extremidade de destino. Com o marcador canônico (`refX=7`, ponta do polígono em `x=8`) e largura de traço do anel `1.2`, `marker_overhang = 1.2`:

```text
phi_entry = atan2(q_entry(j).y - cy, q_entry(j).x - cx)
phi_end   = phi_entry - marker_overhang / R
q_end     = C + R * (cos(phi_end), sin(phi_end))

M q_exit(k).x q_exit(k).y
A R R 0 0 1 q_end.x q_end.y
```

A flag large-arc é `0` porque os intervalos entre estações adjacentes são menores que 180 graus; a flag sweep é sempre `1` para movimento horário em coordenadas SVG. O avanço da ponta da seta completa o 1.2px final até `q_entry(j)`, caindo sobre a borda da caixa sem atravessar seu traço. O conector de fechamento da estação `N-1` para a estação 0 usa a fórmula idêntica.

Os arcos circulares do anel do Loop são uma exceção documentada, específica deste tipo, a SKILL.md §6 regra 1, seguindo o mesmo precedente dos arcos de promoção do Medallion. Um Loop nunca mistura segmentos cúbicos, retos ou ortogonais arredondados no seu anel: os seis intervalos visíveis devem ser lidos como pedaços de um único círculo contínuo.

### 2.3 Extremidades dos raios tracejados de write-back

Cada raio vai para dentro, da borda da estação em direção ao hub. Use a mesma fórmula de interseção raio/caixa, agora sobre o vetor radial `u_k`:

```text
box_distance(v, half_w, half_h) = min(half_w / abs(v.x), half_h / abs(v.y))
                                        # ignore a term whose denominator is zero

d_station = box_distance(u_k, a, b)
d_hub     = box_distance(u_k, A, B)
marker_gap = 6                         # 4..8px; 6px canonical

spoke_start(k) = P_k - d_station * u_k
hub_edge(k)    = C   + d_hub     * u_k
spoke_end(k)   = C   + (d_hub + marker_gap) * u_k
```

Como a seta vai da estação em direção a `C`, somar `marker_gap` deixa a extremidade logo fora do limite do hub. A ponta de seta mais leve para antes do traço do hub em vez de colidir com ele. Os raios radiais são a exceção específica deste tipo à proibição geral de conectores retos inclinados; eles devem permanecer raios verdadeiros, não devem cruzar uns aos outros e só podem tocar sua estação de origem e o hub.

Os rótulos são opcionais quando o sublabel da estação já nomeia o write-back. Quando usados, seguem o papel `arrow-label`, ficam de um lado do raio e recebem uma máscara `paper` opaca com um espaço visível de 6–10px em relação ao traço. Rotule um subconjunto selecionado em vez de forçar seis rótulos no halo do hub.

### 2.4 Dimensionamento do viewBox

O *viewBox* deve incluir os retângulos completos das estações, as curvas externas do anel, as pontas de seta e pelo menos `margin` de respiro:

```text
left   <= cx - R - station_w/2 - margin
right  >= cx + R + station_w/2 + margin
top    <= cy - R - station_h/2 - margin
bottom >= cy + R + station_h/2 + margin

viewBox_w = right - left
viewBox_h = bottom - top
```

Inclua os extremos completos do círculo `cx +/- R`, `cy +/- R`, mais os limites das estações e a folga dos marcadores, ao verificar esses limites. Nunca reduza o canvas a ponto de cortar um traço de estação, um marcador ou um arco do anel. Para o exemplo canônico de seis estações, `viewBox="0 0 1040 680"`, `C=(520,340)`, `R=240`, tamanho de estação `160×64` e tamanho de hub `200×104` deixam uma folga externa generosa.

---

## 3. Gramática visual

| Elemento | Tratamento |
|---|---|
| Estação | Nó padrão: preenchimento `paper`, traço `ink`, `radius-md`; nome em `node-name`, sublabel em `sublabel` |
| Hub | O único elemento escuro: preenchimento `ink`, texto `paper`; ligeiramente maior que uma estação |
| Estação focal | No máximo uma: preenchimento `accent-tint`, traço `accent`; o nome da estação pode usar `accent` |
| Fluxo do anel | Arcos circulares `A R R 0 0 1` sobre o círculo das estações, traço sólido `muted`, ponta de seta padrão no destino; somente sentido horário |
| Raio de write-back | Traço tracejado `soft` com ênfase reduzida, `stroke-dasharray="5,4"`, com ponta de seta `soft` |
| Rótulo do raio | Papel `arrow-label`, `soft`, maiúsculas, máscara `paper`, 6–10px afastado do conector |

Desenhe nesta ordem: `paper` ou grade de pontos opcional → setas do anel → raios tracejados → máscaras e rótulos dos raios → caixas das estações → hub → texto. Os nós mascaram ultrapassagens microscópicas dos conectores, enquanto toda extremidade pretendida ainda cai sobre uma borda.

O hub não é uma sétima etapa do processo. Ele é estado acumulado: memória, padrões, evidências, políticas ou um registro operacional compartilhado. Limite seu texto a um nome mais um sublabel curto.

---

## 4. Regras de conectores (obrigatórias)

SKILL.md §6 se aplica integralmente, exceto pelos dois primitivos de conector específicos do Loop: arcos circulares do anel (§2.2) e raios radiais retos (§2.3). Assim como os arcos de promoção do Medallion, estes substituem §6 regra 1 para este tipo de diagrama:

- As setas do anel são arcos circulares de mesmo raio, sólidas e no sentido horário. Todo caminho usa `A R R 0 0 1`; os marcadores de destino caem sobre as bordas das estações e nenhum conector termina em um ponto central.
- Os raios são tracejados e apontam para dentro. Um raio sólido destrói a distinção visual entre fluxo operacional e write-back.
- Os rótulos usam máscaras opacas e mantêm um espaço visível de 6–10px em relação ao conector. Nunca coloque texto sobre o traço.
- Nenhum conector do anel ou raio pode se sobrepor a outro conector. Os caminhos do anel ficam fora do hub; os raios ocupam rotas radiais distintas.
- Quando dois raios precisarem sair da mesma borda de estação, distribua (fan-out) seus pontos de conexão pela fórmula de §6, com pelo menos 12px de separação. O Loop normal tem um raio por estação; use um segundo somente quando a semântica não puder ser mesclada.
- Se uma rota do anel fosse cruzar o hub, aumente `R` ou divida o diagrama. Não passe o fluxo pelo estado compartilhado nem substitua por uma rota ortogonal.

---

## 5. Variante escura — troca de tokens

Aplique a regra de inversão do style guide; não invente uma segunda paleta.

| Papel | Claro | Escuro |
|---|---|---|
| Canvas e preenchimento das estações | `paper` | `paper` escuro |
| Texto principal e traço das estações | `ink` | `ink` invertido |
| Preenchimento do hub / texto do hub | `ink` / `paper` | `ink` invertido / `paper` escuro |
| Fluxo do anel | `muted` | `muted` escuro |
| Raios de write-back e rótulos | `soft` | `soft` escuro |
| Preenchimento / traço focal | `accent-tint` / `accent` | `accent-tint` escuro / `accent` escuro mais brilhante |
| Linha e grade de pontos | `rule` | `rule` invertido com a mesma opacidade |

A relação semântica permanece inalterada no modo escuro: um hub preenchido com `ink`, uma estação `accent` opcional, setas sólidas neutras no anel e write-backs tracejados mais leves.

---

## 6. Checklist de reprodutibilidade

1. A quantidade de estações é 5–8 e a quantidade de hubs é exatamente um.
2. A estação 0 está em `-90deg`; todas as outras usam passos iguais de `360/N` no sentido horário.
3. Toda seta sólida do anel conecta estações adjacentes com `A R R 0 0 1`, usando o mesmo `R`; a última retorna à primeira.
4. Todo marcador do anel cai sobre a borda de uma estação, não sobre seu centro.
5. Todo raio tracejado começa na borda interna de uma estação e para `marker_gap` antes do traço do hub.
6. Os conectores do anel ficam fora do hub; os raios não se cruzam nem se sobrepõem.
7. No máximo uma estação usa `accent-tint` + `accent`; somente o hub usa o preenchimento escuro `ink`.
8. Os rótulos dos raios, se presentes, usam `arrow-label`, uma máscara opaca e um espaço de 6–10px.
9. O *viewBox* inclui as caixas das estações, traços, curvas, marcadores e margens sem cortes.

---

## 7. Antipadrões

| Antipadrão | Por que falha / correção |
|---|---|
| Dois hubs | Dois estados acumulados criam dois sistemas. Desenhe dois diagramas. |
| Raios sólidos | Parecem fluxo principal e eliminam o sinal tracejado de retorno. Use write-backs tracejados `soft`. |
| Estações em ângulos desiguais sem motivo | O anel deixa de ser lido como uma única cadência operacional. Use espaçamento igual de `360/N`, a menos que um agrupamento de fases documentado exija um intervalo deliberado. |
| Segmentos de anel mistos arco + ortogonais | O anel vira um retângulo arredondado. Todo segmento deve ser um arco circular de mesmo raio, para que o anel seja lido como um único círculo contínuo. |
| Conectores cruzando o hub | O fluxo se confunde com o estado. Passe o anel por fora ou aumente o raio. |
| Accent em várias estações | O ponto de decisão editorial desaparece. Mantenha no máximo uma estação focal. |
| Mais de 8 estações | Rótulos e raios congestionam o hub. Divida em visão geral + detalhe. |
| Um ciclo que nunca retorna de fato | Isso é um Flowchart disposto em círculo. Use Flowchart e mostre o ponto final real. |

---

## 8. Exemplos

- `assets/example-loop.html` — mínimo claro: loop operacional de autoaperfeiçoamento com seis estações.
- `assets/example-loop-dark.html` — a mesma geometria sob a inversão de tokens escura.
- `assets/example-loop-full.html` — página editorial com o loop principal, três cartões de resumo e colofão.
