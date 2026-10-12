# Janela de terminal (variante com chrome de CLI)

Skin (tema) opcional de página inteira que envolve qualquer diagrama em uma janela de terminal falsa — barra de título com três pontos, uma linha de prompt `$`, tipografia monoespaçada em tudo. Use para anúncios de ferramentas de desenvolvimento, posts de produtos CLI e cards técnicos para redes sociais em que uma captura de tela precisa ser lida como "terminal", não como "documento editorial".

Esta é uma **segunda skin, fixa** — veja [style-guide.md § Terminal skin](style-guide.md#terminal-skin-opt-in-alternate) para a tabela de tokens. Ela não herda os tokens de marca de `onboarding.md` e não faz parte da regra de inversão claro/escuro; todo exemplo de terminal usa os mesmos nove tokens, independentemente da marca do site anfitrião.

## Gramática

```html
<div class="terminal">
  <div class="titlebar">
    <div class="dot accent"></div>
    <div class="dot"></div>
    <div class="dot"></div>
    <div class="titlebar-name">loop.sh — self-improving-loop</div>
  </div>
  <main class="frame">
    <p class="prompt">
      <span class="sign">$</span> diagram-design render --type loop
    </p>
    <h1># The self-improving loop</h1>
    <svg>...</svg>
  </main>
</div>
```

```css
body {
  background: var(--terminal-page);
}
.terminal {
  background: var(--terminal-paper);
  border: 1px solid var(--terminal-border);
  border-radius: 12px;
}
.titlebar {
  background: var(--terminal-bar);
  border-bottom: 1px solid var(--terminal-border);
}
.dot {
  background: var(--terminal-soft);
}
.dot.accent {
  background: var(--terminal-accent);
}
```

Dentro do SVG, troque os tokens padrão claro/escuro 1:1 pelos equivalentes `terminal-*`: `paper` → `terminal-paper`, `ink` → `terminal-ink`, `muted`/`soft` → `terminal-muted`/`terminal-soft`, `accent`/`accent-tint` → `terminal-accent`/`terminal-accent-tint`. O padrão de hub/nó focal (preenchimento invertido para o único elemento destacado) continua valendo.

## Tipografia

**Tudo é monoespaçado** — esta é a única variante em que isso é correto. Abandone o Amazon Ember por completo; defina o título da página em mono, negrito, prefixado com `# ` (lido como uma linha de comentário). O eyebrow vira um prompt de shell: `$ ` em `terminal-accent`, o comando em `terminal-muted`.

Use todos os papéis de texto **1–2px acima** da escala tipográfica padrão em `style-guide.md` (ex.: `node-name` 12px → 14px, `sublabel`/`arrow-label` 8–9px → 9–10px, rótulo do hub 16px → 18px). Monoespaçada nos tamanhos padrão parece pequena ao lado da mistura sans/serif que está substituindo, e esses cards geralmente são vistos na escala de feed de rede social, não em tela cheia.

## Pontos da barra de título

Três círculos de 10px, no estilo macOS. A **regra de 1 accent limita o uso de cor aqui também**: um ponto é `terminal-accent`, os outros dois são `terminal-soft`. Não use a tríade de semáforo vermelho/amarelo/verde — isso seria uma segunda e uma terceira matiz, o que a paleta proíbe.

## Regras críticas

- Nada de preto puro (`#000000`) — use `terminal-page` (`#0a0a0a`) / `terminal-paper` (`#141414`). Mesma regra da skin padrão, mesmo motivo: o preto absoluto estoura em telas OLED e na impressão.
- Apenas um accent. Se um diagrama precisar de um segundo elemento focal, use `terminal-ink` (branco) para dar ênfase via peso/tamanho, não uma segunda cor.
- O padrão de grade de pontos no fundo (se usado) permanece `rgba(255,255,255,0.06–0.08)` — uma textura quase invisível, não um concorrente visual do chrome da barra de título.

## Quando usar

- Posts de lançamento de ferramentas de desenvolvimento / produtos CLI (pacote npm, flag de CLI, fluxo de trabalho baseado em terminal).
- Cards técnicos para redes sociais em que "esta é uma ferramenta para engenheiros" faz parte da mensagem.
- Capturas de tela feitas para se destacar em um feed com muito modo escuro (X, Discord, blogs de desenvolvimento).

## Quando não usar

- Posts editoriais / de formato longo — combine com a variante padrão clara ou editorial completa.
- Saída alinhada à marca a partir de `onboarding.md` — terminal é uma skin fixa, não tokenizada por marca. Não tente conciliar as duas.
- Qualquer diagrama cujo público não esteja acostumado, como desenvolvedor, a ler `$`/`#`/pontos da barra de título como chrome em vez de conteúdo.
