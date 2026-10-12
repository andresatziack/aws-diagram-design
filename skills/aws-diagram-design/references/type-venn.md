# Venn / Sobreposição de conjuntos (Venn / Set Overlap)

**Ideal para:** interseção de conceitos/domínios, atributos compartilhados entre categorias, "onde A encontra B", quadros no estilo ikigai (desejável × viável × sustentável).

## Convenções de layout
- **Prefira 2 ou 3 círculos.** Evite 4+ (ilegível — use uma matriz).
- Traço do círculo: linha fina (hairline) de 1px, cor por conjunto (ink, muted, soft).
- Preenchimento do círculo: tom de opacidade muito baixa — `rgba(35,47,62,0.04)` para o conjunto ink, `rgba(84,91,100,0.05)` para muted. Os tons se somam naturalmente nas regiões de sobreposição.
- Raios: iguais quando os conjuntos têm tamanho comparável; proporcionais quando os conjuntos são significativamente diferentes. Não finja tamanhos iguais por estética.
- **Rótulos de conjunto** posicionados fora do círculo, NUNCA cruzando o traço. Amazon Ember 12–14px 600 para o nome do conjunto, sublabel opcional em Amazon Ember Mono 9px.
- **Rótulos de interseção** posicionados dentro da região de sobreposição, Amazon Ember 12px 600, centralizados. Para sobreposições pequenas, use uma linha guia até um rótulo em espaço livre.
- **Accent coral** em UMA interseção focal — o "ponto ideal" (sweet spot). Ou traço coral no rótulo OU preenchimento com tom coral limitado por clipPath (`rgba(255,153,0,0.10)`).
- Centros e raios dos círculos divisíveis por 4.

## Antipadrões
- Regiões sem rótulo — o leitor não consegue distinguir qual conjunto é qual.
- Círculos que não se sobrepõem quando a sobreposição é o objetivo.
- Círculos de tamanho igual quando os conjuntos são obviamente diferentes (desonesto).
- Coral em múltiplas regiões de sobreposição (o sinal focal morre).
- Rótulos posicionados em cima dos traços dos círculos (ilegível).
- 4+ círculos quando 2–3 bastariam.

## Exemplos
- `assets/example-venn.html` — minimal claro
- `assets/example-venn-dark.html` — minimal escuro
- `assets/example-venn-full.html` — editorial completo
