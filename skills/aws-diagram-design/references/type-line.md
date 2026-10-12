# Gráfico de linhas (Line Chart)

**Ideal para:** tendências contínuas ao longo do tempo ou de um índice sequencial — cadastros ao longo das semanas, receita por mês, latência ao longo das releases. Use quando a direção e a taxa de variação entre os pontos são a mensagem principal.

## Convenções de layout

- **Margens da área de plotagem:** esquerda 80px, base 60px, topo 40px, direita 40px — dentro do viewBox `0 0 1000 500`.
- **Pontos:** 4–12 pontos de dados. Menos → considere uma estatística-resumo; mais → agregue em períodos.
- **Eixo x:** rótulos de tempo/índice uniformemente espaçados abaixo da plotagem. Use Amazon Ember Mono 8px, centralizado no x de cada ponto.
- **Linhas de grade do eixo y:** 4–6 horizontais em intervalos regulares. Mesmo tratamento tênue do gráfico de barras.
- **Linhas:** `<polyline>` com `fill="none"`. Série focal `stroke-width="1.8"`, as demais `"1.2"`.
- **Pontos de vértice:** apenas na série focal (`r=4`, preenchidos). Demais séries: apenas a linha.
- **Preenchimento de área (opcional):** `<polygon>` fechando de volta em `y=420` (linha de base do eixo x) com opacidade 0.08. Use apenas na série focal, quando o significado da área for importante.
- **Múltiplas séries:** até 5 séries. Focal = `accent`. As demais = `series-1`, `series-2`, `series-3`, `series-4` de style-guide.md. Aplique a paleta de séries em ordem — não pule nenhuma.
- **Legenda:** faixa horizontal na parte inferior. Amostra = retângulo de 16×8px com o preenchimento/traço da série. Uma entrada por série.

### Padrão de polyline

```svg
<!-- Focal series -->
<polyline points="x0,y0 x1,y1 x2,y2 ..."
          fill="none" stroke="#EC7211" stroke-width="1.8" stroke-linejoin="round"/>
<!-- Dots at each point (focal only) -->
<circle cx="x0" cy="y0" r="4" fill="#EC7211"/>

<!-- Non-focal series -->
<polyline points="x0,y0 x1,y1 ..."
          fill="none" stroke="#7c8f6f" stroke-width="1.2" stroke-linejoin="round"/>
```

## Antipadrões

- Mais de 5 séries (massa visual — reduza ou divida).
- Linhas que não começam em uma linha de base zero compartilhada, a menos que isso seja explicitamente anotado.
- Curvas suavizadas/spline quando os dados subjacentes são amostrados — a polyline é honesta.
- Pontos em todas as séries quando há 4+ séries (apenas a focal recebe pontos).
- Eixo y que não inclui o zero quando a magnitude absoluta importa.
- Conectar segmentos de dados descontínuos sem uma lacuna visual.

## Exemplos

- `assets/example-line.html` — minimal claro
- `assets/example-line-dark.html` — minimal escuro
- `assets/example-line-full.html` — editorial completo
