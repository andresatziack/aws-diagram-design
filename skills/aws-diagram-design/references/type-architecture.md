# Arquitetura

**Ideal para:** visões gerais de sistemas, diagramas de fluxo de dados, mapas de integração, topologia de infraestrutura.

## Convenções de layout
- Agrupe componentes por camada ou fronteira de confiança (frontend → backend → dados; público → privado).
- O fluxo principal corre da esquerda→direita ou de cima→baixo. Escolha um e mantenha-o.
- Desenhe as setas antes das caixas, para que a ordem z coloque as conexões atrás dos componentes.
- 1–2 nós focais em coral: o ponto de integração principal, o armazenamento de dados principal ou o nó de decisão-chave.
- Retângulos de fronteira tracejados marcam regiões (VPC, security group, zona de confiança); os rótulos ficam sobre uma máscara da cor do paper sobre a linha da fronteira.

## Estilo dos conectores

**Conectores em ângulo reto arredondado (ortogonais) são OBRIGATÓRIOS** para todas as conexões que não sejam horizontais/verticais — `<line>` diagonal entre nós fora do mesmo eixo é reprovação imediata (veja SKILL.md §6 Regras obrigatórias de conectores). Caminho em cotovelo com duas dobras e `r=8`:

```svg
<!-- right+down: from (x1,y1) to (x2,y2), mid = (x1+x2)/2 -->
<path d="M x1,y1 H mid-8 Q mid,y1 mid,y1+8 V y2-8 Q mid,y2 mid+8,y2 H x2"
      fill="none" stroke="…" stroke-width="1.2" marker-end="url(#arrow)"/>
```

Inverta os sinais verticais para direita+cima. Use um `<line>` simples apenas quando as extremidades compartilham o mesmo x ou y. Os rótulos das setas ficam no segmento vertical, centralizados horizontalmente em `mid` e verticalmente entre os dois cantos.

**Seleção de portas — use topo/base para conectores verticais.** Quando o destino está visivelmente acima ou abaixo da origem, saia pela borda superior/inferior da origem e entre pela borda superior/inferior do destino. Use um caminho em L com uma única dobra (horizontal → canto → vertical entrando no nó), não uma porta lateral esquerda/direita:

```svg
<!-- entering a node from its bottom (destination above source) -->
<path d="M x1,y_src H x2-8 Q x2,y_src x2,y_src-8 V y_dst"
      fill="none" stroke="…" stroke-width="1.2" marker-end="url(#arrow)"/>
```

Reserve as portas esquerda/direita para conexões que percorrem principalmente a horizontal. Entrar em um nó pela lateral em um caminho predominantemente vertical dá a impressão de que a seta perfura a face do nó, em vez de chegar por cima ou por baixo.

**Caminhos tracejados — mesmas regras de roteamento.** Fluxos opcionais, de retorno, assíncronos e passivos usam `stroke-dasharray="4,3"` e um traço mais leve (`stroke-width="1"`). Aplique as **mesmas regras de roteamento ortogonal, seleção de portas e bridge/hop** dos caminhos sólidos — o padrão tracejado comunica apenas peso semântico, não uma gramática de roteamento diferente. Quando um caminho tracejado e um sólido precisam se cruzar, faça a ponte (bridge) no tracejado (ele é, por definição, a conexão menos importante).

**Margem do rótulo de zona.** Deixe ≥16px entre a base do rótulo eyebrow da zona e o topo do primeiro nó contido. Dimensione o retângulo da zona com altura suficiente para conter esse espaço de cabeçalho (`y` da zona = node_top − 32; `y` da máscara do rótulo = zone_y + 4).

## Setas cruzadas (Crossing arrows) — bridge / hop

Quando duas setas ortogonais precisam se cruzar, adicione um pequeno arco (hop/bridge) na seta **menos importante** no ponto de cruzamento. A seta mais importante é desenhada sem interrupção.

```svg
<!-- Horizontal hop over a vertical crossing at x=cx, on a line at y -->
<path d="M x1,y H cx-8 a 8,8 0 0,1 16,0 H x2"
      fill="none" stroke="…" stroke-width="1.2" marker-end="url(#arrow)"/>
```

`a 8,8 0 0,1 16,0` é um arco SVG: rx=ry=8, large-arc=0, sweep=1 (curva visualmente para cima), avançando 16px para a direita — criando uma saliência semicircular de raio 8px sobre o cruzamento. Para um hop vertical sobre uma horizontal, use `a 8,8 0 0,0 0,16` no caminho vertical.

Decida qual seta receberá a ponte: faça a ponte na que for semanticamente menos importante (passiva, secundária, write-back) ou na que tiver o traço mais leve (tracejada, muted). Nunca faça a ponte em ambas.

## Agrupamento em zonas

Agrupe 2+ nós que servem à mesma camada ou fronteira de confiança com um retângulo de zona — desenhado **antes** das setas e dos nós (ordem z: fundo → zonas → setas → nós):

```svg
<rect x="{x}" y="{y}" width="{w}" height="{h}" rx="8"
      fill="rgba(35,47,62,0.02)" stroke="rgba(35,47,62,0.10)" stroke-width="0.8"/>
<rect x="{label_x}" y="{y+4}" width="{label_w}" height="12" rx="2" fill="{paper}"/>
<text x="{label_cx}" y="{y+13}" fill="rgba(35,47,62,0.40)" font-size="7"
      font-family="'Amazon Ember Mono', ui-monospace, monospace" text-anchor="middle" letter-spacing="0.14em">LAYER</text>
```

Regras:
- Deixe 12–16px acima do primeiro nó contido — o rótulo eyebrow fica nessa margem.
- Preenchimento da zona: `rgba(35,47,62,0.02)` (véu de 2% de ink). Qualquer coisa mais forte compete com o preenchimento dos nós.
- Máximo de 3 zonas por diagrama. Mais do que isso e passa a parecer um swimlane (use esse tipo).
- Modo escuro: troque `rgba(35,47,62,…)` → `rgba(255,255,255,…)` com as mesmas opacidades; preenchimento da máscara do rótulo = `paper` (escuro).

## Antipadrões
- Todas as caixas em coral ("isto também é importante") — a hierarquia desmorona.
- Seta bidirecional quando uma direção é óbvia pelo contexto.
- Legenda flutuando dentro da área do diagrama.

## Exemplos
- `assets/example-architecture.html` — minimal claro
- `assets/example-architecture-dark.html` — minimal escuro
- `assets/example-architecture-full.html` — editorial completo
