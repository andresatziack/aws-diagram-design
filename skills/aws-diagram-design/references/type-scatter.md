# Gráfico de dispersão (Scatter Plot)

**Ideal para:** correlação e distribuição — duas variáveis contínuas plotadas uma contra a outra. Use quando a relação (ou a ausência dela) entre as variáveis é a mensagem, ou quando você precisa identificar agrupamentos, outliers e casos de alto/baixo desempenho.

## Convenções de layout

- **Margens da área de plotagem:** esquerda 80px, base 60px, topo 40px, direita 40px — dentro do viewBox `0 0 1000 500`.
- **Quantidade de pontos:** 5–30 pontos. Menos → apenas descreva a relação em texto; mais → agrupe em um contorno de densidade.
- **Eixos:** X em y=420 (linha de base), Y em x=80. Ambos usam rótulos de linhas de grade em Amazon Ember Mono 8px. 4–6 linhas de grade por eixo, em intervalos iguais.
- **Forma do ponto:** `<circle>` r=5 para pontos padrão, r=6 para o focal. Ponto focal com preenchimento `accent`. Os demais com preenchimento `muted @ 0.20` + traço `muted`.
- **Rótulos nos pontos (opcional):** Amazon Ember Mono 8px ao lado de um ponto. Use uma máscara retangular com preenchimento paper atrás do rótulo. Rotule no máximo 2–3 pontos; não todos.
- **Linha de tendência (opcional):** `<line>` do canto inferior esquerdo ao superior direito, traço `rgba(35,47,62,0.25)` tracejado 4,3. Nunca force um ajuste perfeito — só adicione se a tendência for visualmente óbvia.
- **Divisores de quadrante (opcional):** linhas tracejadas leves na mediana de x e de y para dividir em quadrantes. Rotule cada quadrante em Amazon Ember Mono 8px, muted.

### Padrão de ponto

```svg
<!-- Non-focal point — paper mask + circle -->
<circle cx="X" cy="Y" r="5" fill="#ffffff"/>
<circle cx="X" cy="Y" r="5" fill="rgba(84,91,100,0.20)" stroke="#545B64" stroke-width="1"/>

<!-- Focal point -->
<circle cx="X" cy="Y" r="6" fill="#ffffff"/>
<circle cx="X" cy="Y" r="6" fill="rgba(255,153,0,0.15)" stroke="#EC7211" stroke-width="1.2"/>
```

## Antipadrões

- Mais de 30 pontos sem agrupamento (vira ruído/massa disforme).
- Linha de tendência forçada quando os dados estão genuinamente dispersos — é desonesto.
- Rótulos em todos os pontos (rotule apenas o focal e 1–2 outliers relevantes).
- Codificação por tamanho de bolha (use um rótulo de terceiro eixo ou cor; a percepção de área de bolha não é confiável).
- Eixos que não incluem o zero quando a posição absoluta importa; eixos que incluem o zero quando a faixa é minúscula e distante de zero.

## Exemplos

- `assets/example-scatter.html` — minimal claro
- `assets/example-scatter-dark.html` — minimal escuro
- `assets/example-scatter-full.html` — editorial completo
