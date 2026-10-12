# Quadrante (Quadrant)

**Melhor para:** priorização (Impacto × Esforço), posicionamento (Alcance × Frequência), mapas de portfólio, quadros de decisão 2×2.

## Convenções de layout
- Grade 2×2. Linhas de eixo: cruz de ink de 1px passando pelo centro.
- **Rótulos dos eixos: minimalismo estilo Jobs.** Uma única palavra em cada ponta de seta — nenhum glifo embutido no rótulo (sem `↑` / `→` / `←` / `↓`), sem parênteses, sem modificadores "HIGH / LOW". Amazon Ember Mono 9px, peso regular, espaçamento 0.18em, maiúsculas. Ladeie as pontas das setas — nunca coloque rótulos sobre a linha do eixo. Encurte a seta o suficiente (~60–80px para dentro da borda do viewBox) para deixar respiro para os rótulos além das pontas.
- Nunca rotule no ponto médio.
- Itens: pequenos pontos rotulados (`r=4`) posicionados nos quadrantes. Rótulos a 8–10px de distância; não deixe os rótulos cruzarem as linhas dos eixos.
- Coral no item "fazer primeiro" (tipicamente no canto superior direito).
- Limite a ~12 itens; agrupe ou divida além disso.

## Antipadrões
- Quatro quadrantes preenchidos com cores diferentes — posição + rótulo fazem o trabalho; o ruído de cor o enfraquece.
- Itens posicionados sobre as linhas dos eixos (quadrante ambíguo).
- Nomes dos eixos ausentes.

## Exemplos
- `assets/example-quadrant.html` — minimal claro
- `assets/example-quadrant-dark.html` — minimal escuro
- `assets/example-quadrant-full.html` — full editorial
- `assets/example-quadrant-consultant.html` — consultant special (veja abaixo)

---

## Consultant special (2×2 scenario matrix)
*Especial de consultoria (matriz de cenários 2×2)*

Uma **variante de layout** do quadrante padrão — mesma skin (tema) da casa (paper quente, padrão de pontos, título em Amazon Ember (700), sobretítulos (eyebrows) em Ember Mono, regra focal em coral). A gramática muda: os eixos representam uma **faixa** em vez de uma medida; as células contêm **cenários nomeados** em vez de itens posicionados.

**Use quando:** você estiver enquadrando quatro futuros, arquétipos ou opções estratégicas ao longo de dois vetores independentes — planejamento de cenários clássico, quadros de posicionamento ou apresentações de estratégia 2×2 (território BCG/McKinsey). O leitor deve sair com quatro apostas nomeadas, não com uma nuvem de pontos.

**Não use** para priorização, mapas de densidade ou qualquer coisa em que a *posição dentro* de uma célula carregue significado — esse é o quadrante padrão acima.

### O que a torna a variante de consultoria

| Movimento | Quadrante padrão | Consultant special |
|---|---|---|
| Setas dos eixos | uma ponta | **duas pontas** — ambos os eixos têm `marker-start` + `marker-end` |
| Conteúdo da célula | pequenos pontos com rótulos | **cenário nomeado + descrição de 1–3 linhas** |
| Canto do quadrante | etiqueta curta (ex.: DO FIRST) | **etiqueta numerada + combinação dos eixos** (`01 · DIMENSION-A / DIMENSION-B`) |
| Destaque focal | coral em um *item* | coral em um *quadrante* — fundo tingido + traço coral + etiqueta de canto coral |
| Eixos | ink muted de 1px | **ink de 1.2px** (um pouco mais pesado — os eixos carregam mais da figura) |

Ambas as variantes usam os mesmos rótulos de eixo minimalistas estilo Jobs: uma palavra em cada ponta de seta, sem glifos, sem parênteses. A única diferença nos eixos é que a variante de consultoria usa setas com duas pontas em vez de uma.

Todo o resto — paper, padrão de pontos, tipografia, faixa de legenda, grade de 4px, orçamento de complexidade — é o padrão da casa. Não invente novas cores ou fontes para esta variante.

### Tokens de estilo (da casa)

- **Paper / fundo / padrão**: padrões do `style-guide.md` (`paper`, padrão de pontos 22×22 a 10% de ink).
- **Linhas dos eixos**: `ink` (`#232F3E`), `stroke-width: 1.2`, `marker-start` + `marker-end` ambos apontando para fora.
- **Tingimento do quadrante focal**: `rgba(255,153,0,0.04)` em um retângulo completo atrás da célula focal.
- **Célula focal**: preenchimento `accent-tint`, traço `accent` de 1.2px. Etiqueta de canto em `accent`, peso 600.
- **Células não focais**: tratamento `store` (preenchimento `ink @ 0.04`, traço `muted @ 0.28`).
- **Título da célula**: Amazon Ember, 16px, peso 600, `ink`.
- **Descrição da célula**: Amazon Ember, 11px, `muted`, 1–3 linhas, alinhada à esquerda dentro da célula.
- **Etiqueta de canto**: Amazon Ember Mono, 8px, maiúsculas, espaçamento `0.18em`, `muted` (ou `accent` na focal). Formato: `NN · DIMENSION-A / DIMENSION-B` — as duas palavras de dimensão dos eixos devem corresponder exatamente aos rótulos dos eixos.
- **Rótulos dos eixos**: Amazon Ember Mono 9px **peso regular** (não negrito), espaçamento `0.18em`, maiúsculas, `ink`. **Uma palavra por ponta.** Sem glifos de seta no rótulo, sem parênteses `HIGH / LOW`, sem sublabels de várias linhas. A própria palavra *é* o rótulo. Posicione os rótulos *além* das pontas das setas (não sobre a linha do eixo):
  - Ponta superior: `text-anchor="middle"`, ~12px acima da ponta da seta
  - Ponta inferior: `text-anchor="middle"`, ~20px abaixo da ponta da seta
  - Ponta esquerda: `text-anchor="end"`, ~12px à esquerda da ponta da seta, `dominant-baseline="middle"`
  - Ponta direita: `text-anchor="start"`, ~12px à direita da ponta da seta, `dominant-baseline="middle"`

### Convenções de layout

- Quatro células de tamanho igual (240×160 ou 280×180 são bons padrões), dispostas com um vão de 40–60px a partir da cruz dos eixos.
- A cruz dos eixos passa *entre* as células, não através delas.
- As pontas das setas ficam ~20–40px fora da borda da célula mais externa; os rótulos de eixo de uma palavra ficam ~12px além de cada ponta (veja Rótulos dos eixos acima).
- Exatamente uma célula focal. Não escolher nenhuma a transforma em um template de espaço reservado; escolher duas apaga o sinal.
- Mantenha a faixa de legenda + a linha horizontal na base — igual ao quadrante padrão. As amostras da legenda devem mostrar tanto "aposta principal" (coral) quanto "futuro candidato" (neutro).

### Antipadrões (específicos da variante)

- Fundo branco liso — o paper quente + padrão de pontos é essencial em toda a skill; removê-lo para "parecer de consultoria" torna o diagrama genérico.
- H1 com peso de corpo de texto — mantenha Amazon Ember (700) para o título da página. O contraste título/diagrama é a assinatura da casa.
- Células sem nome ("Cenário 1/2/3/4") em um diagrama entregue — OK como template em branco; não OK como artefato finalizado.
- Coral em mais de uma célula — mesma regra focal de todo o resto da skill.
- Grades 3×3 ou 2×3 — esses são diagramas diferentes, não esta variante.
- Posicionar pontos *dentro* das células — se a posição importa, use o quadrante padrão.
- Rótulos de eixo em negrito, glifos de seta no texto (`↑ DRIVER`) ou parênteses "HIGH / LOW" — todos proibidos. O minimalismo estilo Jobs é inegociável nesta variante.
- Etiquetas de canto que discordam dos rótulos dos eixos (ex.: o eixo diz `REMOTE / IN-PERSON`, mas a etiqueta diz `HIGH REMOTE / LOW AI`). O leitor interpreta isso como um bug em três segundos.
