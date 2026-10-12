# Radar / Teia (Spider)

**Melhor para:** comparar 3–5 entidades em 3–5 critérios quantitativos em uma única escala normalizada 0–N. Matrizes de capacidades, avaliações de produtos ou backends, scorecards de frameworks/times. Onde uma tabela comparativa começa a ficar sem espaço horizontal, o radar torna a forma de cada opção legível num relance.

## Convenções de layout

- **N eixos (3–5).** Igualmente espaçados em um polígono regular de N lados. Primeiro eixo no topo (`-90°`), seguindo no sentido horário. **Acima de 5 → divida ou use uma tabela comparativa.**
- **Cinco anéis concêntricos de grade** nas frações `0.2 / 0.4 / 0.6 / 0.8 / 1.0` do raio. Desenhados como polígonos fechados ligando os vértices dos eixos naquela fração. Os quatro internos em `rule` com opacidade 0.10, o anel externo em `rule-solid` 0.20 (um pouco mais forte, para ancorar o gráfico).
- **Raios dos eixos** do centro até cada vértice externo. `rule-solid` com opacidade 0.20. **Sem pontas de seta.**
- **Rótulos dos eixos:** uma palavra por raio (minimalismo estilo Jobs). Amazon Ember 11px, peso 600. Posicione 16px fora do anel externo, ao longo do vetor do eixo. Topo/base = `text-anchor="middle"`; lado direito = `start`; lado esquerdo = `end`.
- **Marcas de escala** (ex.: `2 4 6 8 10`) somente no **primeiro eixo (o do topo)** — colocar números em todos os raios polui o gráfico rapidamente. Amazon Ember Mono 8px, `muted`, ancoradas no fim em `cx − 6`.
- **Polígono da série:** traço de 1.5px na cor da série, preenchimento na mesma cor com opacidade `0.18` (`0.22` no escuro). Traço de 1.8px na série focal — um aumento sutil de peso.
- **Pontos nos vértices:** **somente na série focal**, `r=4` preenchidos com a cor da série. Séries não focais têm apenas traço e preenchimento. Esta é a regra essencial que mantém o gráfico legível com 4–5 séries.
- **Ordem de desenho:** fundo com padrão de pontos → anéis da grade → raios dos eixos → rótulos dos eixos → marcas de escala → séries não focais (menor área primeiro) → série focal → pontos dos vértices focais → legenda.
- **Legenda:** faixa horizontal na base (conforme a regra global). A amostra é um retângulo 16×8 (correspondendo ao traço+preenchimento do polígono, não um círculo), seguida do nome da entidade. ~140px entre as entradas. Cauda opcional em itálico à direita com a justificativa (`"One coral. Position is the signal — color reserved for the recommended option."`).

## Matemática

Para o eixo `i` (índice a partir de 0) de `N`, valor `v` na escala `S`, centro `(cx, cy)`, raio externo `R`:

```
angle = -π/2 + 2π · i / N
x = cx + (v / S) · R · cos(angle)
y = cy + (v / S) · R · sin(angle)
```

Uma série com valores `[v0, v1, ..., v(N-1)]` torna-se um `<polygon>` com `points="x0,y0 x1,y1 ..."`.

### Referência pré-calculada (N=5, cx=500, cy=240, R=160, S=10, arredondado para inteiros)

| Fração `f` | i=0 (topo) | i=1 | i=2 | i=3 | i=4 |
|---|---|---|---|---|---|
| 0.2 | 500,208 | 530,230 | 519,266 | 481,266 | 470,230 |
| 0.4 | 500,176 | 561,220 | 538,292 | 462,292 | 439,220 |
| 0.6 | 500,144 | 591,211 | 556,317 | 444,317 | 409,211 |
| 0.8 | 500,112 | 622,201 | 575,343 | 425,343 | 378,201 |
| 1.0 | 500,80  | 652,191 | 594,369 | 406,369 | 348,191 |

Para um valor arbitrário `v` no eixo `i`, pegue o deslocamento unitário da linha acima para aquele eixo (ex.: eixo 1: deslocamento `(152, -49)` a partir do centro) e escale por `v/S`. **Grave as coordenadas como inteiros — pixels fracionários renderizam bem em SVG, mas inteiros mantêm o arquivo fácil de ler.**

### Exemplo completo (N=5)

A série `[9, 8, 9, 9, 9]` em uma escala de 0–10 torna-se:

```svg
<polygon points="500,96 622,201 585,356 415,356 363,196"
         fill="rgba(255,153,0,0.18)" stroke="#EC7211" stroke-width="1.8"/>
```

Cada vértice: `center + (v/10) · (outer_i − center)`, arredondado para o pixel mais próximo.

## Paleta de séries

A regra de "1 focal" da skill continua valendo: `accent` é reservado para a série focal, e uma pequena paleta editorial (`series-1` a `series-5`, definida em [`style-guide.md`](style-guide.md)) cobre as séries não focais. Não recorra a cores livres.

| Slot | Token | Claro | Escuro |
|---|---|---|---|
| Focal | `accent` | `#EC7211` | `#FF9900` |
| 1 | `series-1` (sálvia) | `#7c8f6f` | `#9caf8f` |
| 2 | `series-2` (azul-acinzentado) | `#5e7a9b` | `#82a0c0` |
| 3 | `series-3` (mostarda) | `#b8915a` | `#d3ad7a` |
| 4 | `series-4` (marrom-ferrugem) | `#9c6b50` | `#b88670` |
| 5 | `series-5` (ardósia) | `#6e6479` | `#8d8298` |

## Antipadrões

- **Mais de 5 séries** → mingau visual. Divida em dois gráficos (ex.: "melhor em latência" + "melhor em operações") ou mude para uma tabela comparativa.
- **Eixos em escalas nativas inconsistentes** (um 0–100, outro 0–1) sem normalização. **Sempre normalize para 0–N primeiro** — polígonos de radar comparam *formas*, não valores absolutos.
- **Truques de linha de base zero** — começar o anel interno em v=5 para amplificar as diferenças. A grade começa em 0; se as diferenças parecem pequenas, essa é a leitura verdadeira.
- **Pontos em todas as séries.** Somente a focal leva pontos. Adicioná-los a todas as 4–5 séries transforma o gráfico em uma cortina de contas.
- **Radar com 2 séries** — um gráfico de barras comparativo ou uma tabela de 2 linhas é mais claro.
- **Eixos não quantitativos.** Todos os eixos devem ser mensuráveis na mesma escala normalizada. Misturas como "velocidade" + "cor" + "ano" não pertencem a um radar.
- **Rótulos de eixo em fonte mono.** Nomes vão em Amazon Ember (a regra global). Mono é somente para sublabels técnicos.
- **Paleta arco-íris.** Mesmo com os novos tokens `series-*`, você não precisa de todos os 5 em um gráfico — use apenas tantos quantas entidades não focais você tiver.

## Exemplos

- `assets/example-radar.html` — minimal claro. 4 backends de armazenamento × 5 dimensões de carga de trabalho, MinIO como focal.
- `assets/example-radar-dark.html` — minimal escuro, mesmos dados.
- `assets/example-radar-full.html` — full editorial: enquadramento em contêiner + 4 cards (um por backend) com larguras variadas + rodapé.
