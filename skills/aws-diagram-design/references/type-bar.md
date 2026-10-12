# Gráfico de barras / colunas (Bar / Column Chart)

**Ideal para:** comparar quantidades discretas entre categorias ou intervalos de tempo — velocidade de sprint, receita mensal, adoção de funcionalidades, contagens de coortes. Use quando cada categoria tem um único valor numérico e a comparação entre as barras é a mensagem principal.

## Convenções de layout

- **Orientação:** barras verticais (colunas) são o padrão. Barras horizontais são adequadas quando os rótulos das categorias são longos ou quando há mais de 8 categorias.
- **Margens da área de plotagem:** esquerda 80px (rótulos do eixo y), base 60px (rótulos do eixo x), topo 40px, direita 40px — dentro de um viewBox `0 0 1000 500`.
- **Limite de quantidade de barras:** 4–8 barras. Mais de 8 → agrupe em períodos ou divida em dois gráficos.
- **Largura da barra:** ≥ 50% do pitch da coluna (o espaço entre barras nunca deve exceder a barra). Típico: pitch=110px, barra=72px.
- **Linhas de grade do eixo y:** 4–6 linhas horizontais em intervalos regulares. Traço `rgba(35,47,62,0.08)` (bem tênue), 0.8px. Linha de base do eixo x em `rgba(35,47,62,0.25)`, 1px.
- **Rótulos do eixo y:** alinhados à direita, Amazon Ember Mono 8px muted, em x=72 (8px à esquerda da área de plotagem).
- **Rótulos do eixo x:** centralizados abaixo de cada barra, Amazon Ember 11px 600 para os nomes das categorias.
- **Rótulos de valor:** Amazon Ember Mono 8px acima de cada barra. Rótulo da barra focal em accent; os demais em muted.
- **Barra focal:** no máximo 1 barra com preenchimento/traço accent. Todas as outras com preenchimento `muted @ 0.15` + traço `muted`.
- **Linha do eixo y:** `<line>` vertical fina em x=80, de y=40 até y=420.

### Padrão de elemento de barra

```svg
<!-- Opaque paper mask prevents bleed from background -->
<rect x="X" y="Y" width="W" height="H" fill="#ffffff"/>
<!-- Bar body -->
<rect x="X" y="Y" width="W" height="H" fill="rgba(84,91,100,0.15)" stroke="#545B64" stroke-width="1"/>
<!-- Value label above bar -->
<text x="X+W/2" y="Y-8" fill="#545B64" font-size="8" font-family="'Amazon Ember Mono', ui-monospace, monospace" text-anchor="middle">VALUE</text>
```

Barra focal: substitua o preenchimento por `rgba(255,153,0,0.12)`, o traço por `#EC7211` e o preenchimento do rótulo por `#EC7211`.

## Antipadrões

- Mais de 8 barras sem agrupamento (ilegível em escala normal).
- Eixo y truncado (não começando em 0) — distorce a comparação de magnitude.
- Accent em mais de 1 barra ("tudo é importante" = nada é).
- Extrusão 3-D das barras — sem sombras, sem profundidade.
- Rótulos de categoria girados mais de 45°; prefira rótulos curtos ou um gráfico horizontal.

## Variantes

- **Barras agrupadas:** duas barras por categoria, lado a lado. Use `accent` para a série principal e `series-1` para a secundária. Máximo de 2 grupos.
- **Barras empilhadas:** segmentos empilhados até o total. Use `accent` para o segmento focal; tons de muted para os demais. Indique o total no topo de cada pilha.

## Exemplos

- `assets/example-bar.html` — minimal claro
- `assets/example-bar-dark.html` — minimal escuro
- `assets/example-bar-full.html` — editorial completo
