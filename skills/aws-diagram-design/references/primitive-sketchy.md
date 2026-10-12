# Filtro esboçado (Sketchy Filter — variante desenhada à mão)

Filtro de deslocamento opcional que faz cada traço e borda ondular levemente — transforma qualquer variante minimal em um registro "editorial" desenhado à mão, sem alterar o layout. Use quando o diagrama acompanha um ensaio, e não documentação técnica.

## Gramática

```svg
<defs>
  <filter id="sketchy" x="-2%" y="-2%" width="104%" height="104%">
    <feTurbulence type="fractalNoise" baseFrequency="0.02" numOctaves="2" seed="4"/>
    <feDisplacementMap in="SourceGraphic" scale="1.5"/>
  </filter>
</defs>

<!-- Apply to a group wrapping shapes — NOT text -->
<g filter="url(#sketchy)">
  <!-- rects, paths, circles, lines go here -->
</g>

<!-- Text sits OUTSIDE the filtered group — legibility stays crisp -->
<text ...>Labels go here</text>
```

## Ajuste

| Parâmetro | Faixa | Efeito |
|---|---|---|
| `baseFrequency` | 0.01–0.04 | Menor = linhas onduladas preguiçosas; maior = tremidas. 0.02 é o padrão. |
| `numOctaves` | 1–3 | Mais = mais detalhe de ruído. 2 é suficiente. |
| `scale` | 1–6 | 1 quase imperceptível, 1.5 padrão, 2 visível, 4+ caricato. |
| `seed` | inteiro | Troque para obter um padrão aleatório diferente. |

## Regra crítica
Aplique o filtro às formas, NÃO ao texto. Texto com mapa de deslocamento fica ilegível. Estruture seu SVG de forma que o texto fique em um grupo irmão, fora do grupo filtrado.

## Quando usar
- Ensaio / post de blog / newsletter em que o diagrama é o protagonista de uma página narrativa.
- Registro de "esboço de trabalho" — mostrar que algo ainda está em elaboração, não é a arquitetura final.

## Quando não usar
- Documentação técnica (a precisão importa).
- Diagramas com rótulos densos ou alinhamentos justos (o filtro é percebido como ruído).
- Variantes escuras — a ondulação parece um artefato em fundos escuros. Teste antes.
