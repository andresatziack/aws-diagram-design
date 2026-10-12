# Guia de estilo

**A única fonte da verdade para cores, tipografia e tokens.** Todo diagrama se baseia nele — não em valores hex embutidos em outros arquivos de referência. Se quiser mudar a skin (tema) visual do Schematic, altere este arquivo.

**Skin atual: marca AWS** — paper branco, texto Squid-Ink (`#232F3E`), accent Smile-Orange, tipografia Amazon Ember e os AWS Architecture Icons oficiais ([primitive-aws-icons.md](primitive-aws-icons.md)). Isso substitui o padrão editorial original (white-smoke / atomic-tangerine `#eb6c36`); troque esses valores (ou execute [`onboarding.md`](onboarding.md)) e todo novo diagrama herda a nova skin sem mexer em nenhuma lógica específica de tipo.

---

## Tokens

### Papéis semânticos

Todo token é referido pelo seu **papel semântico**, não pelo valor hex. As referências de tipo (`type-*.md`) e o SKILL.md dizem `accent`, não `#EC7211`.

| Papel | Finalidade | AWS (claro) | AWS (escuro) |
|---|---|---|---|
| `paper` | Fundo da página, preenchimento padrão do nó | `#FFFFFF` (branco — fundo oficial de diagramas AWS) | `#161E2D` (dark squid) |
| `paper-2` | Fundo do contêiner do diagrama, preenchimento secundário | `#F2F3F3` | `#232F3E` (squid ink) |
| `ink` | Texto primário, traço primário | `#232F3E` (squid ink) | `#FFFFFF` |
| `muted` | Texto secundário, traço padrão de seta | `#545B64` | `#B6BEC9` |
| `soft` | Sublabels, rótulos de limites | `#7D8998` | `#8D99A8` |
| `rule` | Bordas de linha fina (hairline) | `rgba(35,47,62,0.12)` | `rgba(255,255,255,0.12)` |
| `rule-solid` | Bordas mais fortes, linhas de base | `#D5DBDB` | `rgba(255,255,255,0.25)` |
| `accent` | Focal / no máximo 1–2 por diagrama | `#EC7211` (smile orange, tom escuro) | `#FF9900` (smile orange) |
| `accent-tint` | Preenchimento de caixas com borda accent | `rgba(255,153,0,0.10)` | `rgba(255,153,0,0.14)` |
| `link` | Chamadas HTTP/API, setas externas | `#0972D3` (azul Cloudscape) | `#539FE5` |

> **Origem da paleta da marca:** marca AWS — `squid ink #232F3E`, `smile orange #FF9900`, branco. O `#FF9900` puro fica fraco como traços/texto finos sobre branco, então o `accent` do modo claro usa o tom mais escuro `#EC7211` (o laranja que a própria AWS usa em UI clara); `#FF9900` é o accent no escuro e a base de `accent-tint`. `muted`/`soft`/`rule-solid`/`link` vêm da escala cinza-azulada da AWS/Cloudscape. As cores de identidade de categoria (`#ED7100`, `#8C4FFF`, …) pertencem aos ícones e às bordas de zona — veja [primitive-aws-icons.md](primitive-aws-icons.md) — e **não** são accents editoriais.

> **Observação:** os arquivos HTML de exemplo pré-gerados em `assets/` foram construídos com skins anteriores e não refletem a skin AWS. Novos diagramas produzidos pela skill devem usar os tokens acima.

### Regra de inversão (claro → escuro)

Qualquer `rgba(35,47,62, X)` no claro vira `rgba(255,255,255, X)` no escuro. Mesmas opacidades, RGB invertido. O accent clareia de `#EC7211` para o `#FF9900` puro sobre paper escuro.

### Paleta de séries (apenas tipos de gráfico com várias séries)

Para tipos de gráfico que realmente precisam distinguir várias entidades sobrepostas (atualmente: **radar**), use a escala de categorias da AWS. A regra de "1 focal" continua valendo — `accent` é reservado para a série focal; a paleta abaixo cobre o restante.

| Token | Claro | Escuro | Notas |
|---|---|---|---|
| `series-1` | `#8C4FFF` (roxo) | `#A97FFF` | Série não focal |
| `series-2` | `#01A88D` (verde-azulado) | `#2BC0A8` | Série não focal |
| `series-3` | `#7AA116` (verde) | `#96BE3C` | Série não focal |
| `series-4` | `#C925D1` (magenta) | `#D95CDF` | Série não focal |
| `series-5` | `#DD344C` (vermelho) | `#E5626F` | Série não focal |

Os preenchimentos ficam em opacidade `0.14` no claro e `0.20` no escuro; os traços usam a cor cheia. **Não aplique esses tokens retroativamente a tipos que não são gráficos** — architecture, swimlane etc. continuam usando variantes de muted-ink. A paleta de séries é opcional (opt-in) para diagramas em que formas sobrepostas exigem cores distinguíveis, não uma licença para adicionar cor em outros lugares.

### Terminal skin (opt-in alternate)

Uma paleta autocontida para o primitivo de janela de terminal (veja [primitive-terminal.md](primitive-terminal.md)) — um registro de cromado de CLI para posts sobre ferramentas de desenvolvimento e cards sociais técnicos. Ela não substitui a skin padrão acima e não é afetada pelo onboarding; é uma segunda skin fixa que você ativa (opt-in) por diagrama.

| Token | Hex | Finalidade |
|---|---|---|
| `terminal-page` | `#0a0a0a` | Fundo da página atrás da janela |
| `terminal-paper` | `#141414` | Corpo da janela, preenchimento do nó |
| `terminal-bar` | `#1b1b1b` | Faixa da barra de título |
| `terminal-border` | `#2b2b2b` | Borda da janela, linhas finas (hairlines) |
| `terminal-ink` | `#f5f5f5` | Texto primário, traço primário |
| `terminal-muted` | `#9a9a9a` | Texto secundário, sublabels, traço de anel |
| `terminal-soft` | `#5c5c5c` | Terciário — pontos inativos, raios |
| `terminal-accent` | `#ff5a36` | O único accent — estação focal, sinal de prompt, ponto ativo |
| `terminal-accent-tint` | `rgba(255,90,54,0.12)` | Preenchimento de caixas com borda accent |

**A regra de 1 accent continua valendo.** Tudo o que não for `terminal-ink` ou `terminal-muted`/`terminal-soft` deve ser `terminal-accent` — nunca introduza um segundo matiz.

---

## Tipografia

| Papel | Família | Tamanho | Peso | Uso |
|---|---|---|---|---|
| `title` | Amazon Ember | 1.75rem | 700 | H1 da página |
| `node-name` | Amazon Ember | 12px | 600 | Rótulos legíveis por humanos |
| `sublabel` | Amazon Ember Mono | 9px | 400 | Porta, protocolo, URL, tipo de campo |
| `eyebrow` | Amazon Ember Mono | 7–8px | 500, espaçamento 0.18em, maiúsculas | Tags de tipo, rótulos de eixo |
| `arrow-label` | Amazon Ember Mono | 8px | 400, espaçamento 0.06em | Anotações de setas |
| `callout` | Amazon Ember *itálico* | 14px | 400 | Apenas comentários editoriais |

### Pilha de fontes (Font stack)

Amazon Ember **não está no Google Fonts**, mas esta skill **a inclui** em [`assets/fonts/`](../assets/fonts/) (webfonts woff2 + TTFs para desktop, do download público de tipografia da Amazon). Carregue nesta ordem:

1. **`local('Amazon Ember')`** — presente em máquinas gerenciadas pela Amazon, ou após executar `./install.sh fonts` (raiz do repositório), que instala os TTFs incluídos no diretório de fontes do sistema operacional.
2. **woff2 incluído via `@font-face`** — cole o bloco canônico de [`assets/fonts/README.md`](../assets/fonts/README.md) no HTML gerado, apontando `FONTS` para `<skill-dir>/assets/fonts/woff2` (caminho relativo quando a saída fica perto da skill; ou copie os arquivos woff2 para junto do entregável e use `./fonts/`).
3. **Pilha de fallback** — sempre declarada em `font-family`; visualizadores em sandbox (o renderizador de SVG do GitHub, ferramentas offline) recorrem a Helvetica/Arial.

Mapa de pesos das faces incluídas: Amazon Ember 300 Lt · 400 Rg (+itálico) · 600 SBd · 700 Bd · 800 He; **Amazon Ember Mono 400/700** cobre todos os papéis mono (sublabels, eyebrows, rótulos de seta — uma especificação 500/600 se ajusta ao peso incluído mais próximo, o que é esperado). **Nenhuma fonte é carregada de uma CDN externa** — os diagramas gerados não precisam de `<link>` do Google Fonts.

```css
/* sans (names, titles, callouts) */ font-family: 'Amazon Ember', 'Helvetica Neue', Helvetica, Arial, sans-serif;
/* mono (technical content)       */ font-family: 'Amazon Ember Mono', ui-monospace, monospace;
```

Faces Ember instaladas: Light 300 / Regular 400 / Medium 500 / Bold 700 / Heavy 800, mais itálicos. Se um peso 600 for renderizado como negrito sintetizado em um ambiente de destino, baixe os nomes dos nós para 500 ou suba para 700 — nunca simule negrito na face Heavy.

**Regra estrutural:** mono é para conteúdo *técnico* (portas, comandos, URLs, tipos de campo). Nomes, títulos e callouts vão em Amazon Ember. Ember itálico é reservado para callouts de anotação (veja [primitive-annotation.md](primitive-annotation.md)). **Nunca JetBrains Mono** como fonte "dev" genérica. Esta skin abandona intencionalmente a serifada editorial (Instrument Serif) — a tipografia da marca AWS é de família única.

---

## Traço, raio, espaçamento

| Token | Valor | Uso |
|---|---|---|
| `stroke-thin` | `0.8` | Contornos de caixas de tag, nós folha |
| `stroke-default` | `1` | A maioria dos traços |
| `stroke-strong` | `1.2` | Traços de ênfase |
| `radius-sm` | `4` | Tags pequenas |
| `radius-md` | `6` | Caixas de nós |
| `radius-lg` | `8` | Contêineres, anéis |
| `grid` | `4` | Toda coordenada, tamanho e espaçamento é divisível por 4 (regra rígida) |

---

## Tipo de nó → tratamento

Combinações de papéis semânticos — referencie-as pelo nome nas especificações de tipo.

| Tipo | Preenchimento | Traço |
|---|---|---|
| `focal` (no máximo 1–2) | `accent-tint` | `accent` |
| `backend` | `#ffffff` (branco) | `ink` |
| `store` | `ink @ 0.05` | `muted` |
| `external` | `ink @ 0.03` | `ink @ 0.30` |
| `input` | `muted @ 0.10` | `soft` |
| `optional` | `ink @ 0.02` | `ink @ 0.20` tracejado `4,3` |
| `security` | `#DD344C @ 0.05` | `#DD344C @ 0.60` tracejado `4,4` — alinhado ao vermelho de Security-Identity da AWS |

Os tratamentos de nós e zonas específicos da AWS (nós com ícones de serviço, contêineres de VPC/sub-rede/conta) são especificados em [primitive-aws-icons.md](primitive-aws-icons.md); as cores de identidade das bordas de zona não contam no orçamento (limite) de accent.

---

## Personalizando a skin

Três opções:

1. **Execute o onboarding** — veja [`onboarding.md`](onboarding.md). Informe uma URL; a skill extrai a paleta + fontes e reescreve este arquivo.
2. **Edite à mão** — altere os valores hex nas tabelas acima. Depois, execute o *taste gate* pré-saída para verificar se o accent ainda é lido como "focal" contra a nova cor de paper.
3. **Handoff de marca** — cole o JSON de design tokens existente em uma nova seção aqui e mapeie seus tokens para os papéis semânticos acima.

### Restrições (não as quebre)

- **Contraste**: `ink` deve atingir WCAG AA sobre `paper`. `muted` deve atingir AA sobre `paper` para texto de 11px ou mais.
- **Um accent**: escolha uma cor para `accent`. Dois accents apagam o sinal focal. (As cores de identidade de ícones/zonas da AWS são isentas — são marcas de identidade, não accents.)
- **Sem paleta arco-íris**: se sua marca tem 8 cores, escolha 3 (paper, ink, accent). O restante vira variantes de `muted`.
- **Tipografia**: no máximo três famílias. A skin AWS usa sans + mono (Ember + Amazon Ember Mono) — a consistência da marca pesa mais que o contraste serifada/sans que o padrão editorial usava.
- **Paper**: a skin AWS usa branco puro porque os diagramas de arquitetura oficiais da AWS usam. Para skins não AWS, prefira um neutro quente (creme, osso, cinza-claro) — branco puro sem uma exigência da marca torna o design estéril.
- **Padrão de pontos é opcional, não padrão**: o padrão de pontos 22×22 é uma variante opcional (opt-in) de "papel pontilhado" (boa para diagramas de destaque editoriais longos). O fundo padrão é um preenchimento `paper` limpo, sem padrão. Quando o padrão estiver ativado, ele deve ficar a ~10% de opacidade de `ink` sobre `paper` — visível, mas discreto.
- **O contêiner é limpo por padrão**: o diagrama fica diretamente sobre o paper da página, sem fundo ou borda de contêiner secundário. Uma variante emoldurada (fundo `paper-2` + borda `rule` + raio de 8px + padding) está disponível como opção (opt-in) para layouts com muitos cards, mas não recorra a ela por padrão — o cromado extra compete com a figura.
