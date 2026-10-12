# Callout de anotação (aparte em itálico)

Use para apartes editoriais — o "ponteiro em itálico" que marca um detalhe sem competir com a gramática principal do diagrama. Pense em notas de margem: *"structure IS the index"*, *"no imports, no configuration"*.

## Gramática

```svg
<!-- 1. Italic Amazon Ember text -->
<text x="904" y="36" fill="#232F3E" font-size="14" font-style="italic"
      font-weight="400"
      font-family="'Amazon Ember', 'Helvetica Neue', Helvetica, Arial, sans-serif" text-anchor="end">no imports, no configuration</text>
<!-- 2. Dashed Bézier leader -->
<path d="M 820 44 Q 700 84 520 216" fill="none"
      stroke="rgba(35,47,62,0.40)" stroke-width="1" stroke-dasharray="4,3"/>
<!-- 3. Landing dot -->
<circle cx="520" cy="216" r="2" fill="#232F3E"/>
```

## Regras
- Amazon Ember em itálico sinaliza "voz editorial" em contraste com o corpo sans/mono reto do diagrama. Não substitua por texto reto ou mono em itálico — o itálico é estrutural.
- O caminho tracejado (`stroke-dasharray="4,3"`) distingue a linha guia do callout das setas principais (que são sólidas).
- Posicione os callouts nas margens (canto superior direito, canto inferior esquerdo). Nunca dentro da área ativa do diagrama.
- Máximo de 2 callouts por diagrama. Mais do que isso vira comentário, não sinal.

## Cores

| Intenção | Texto | Linha guia |
|---|---|---|
| Aparte neutro | ink `#232F3E` | `rgba(35,47,62,0.40)` |
| Focal / accent | coral `#EC7211` | `rgba(255,153,0,0.50)` |
| Terciário (muted) | muted `#545B64` | `rgba(35,47,62,0.30)` |

## Antipadrões
- Linha guia em seta sólida (é lida como seta de fluxo).
- Texto reto ou mono em itálico — o itálico é estrutural.
- Callouts cruzando setas principais / linhas de vida — desloque para uma margem livre.
- Usar um callout para rotular algo que o próprio diagrama deveria rotular diretamente — coloque o rótulo no elemento.
