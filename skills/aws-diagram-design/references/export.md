# Exportação para PNG / SVG

Converta um arquivo HTML de diagrama gerado em um `.svg` e/ou `.png` portátil ao lado dele. **Somente manual — nunca execute sem que seja solicitado.**

## Gatilho

Carregue este arquivo quando:

- O usuário invocar `/aws-diagram-design:export-diagram <html-file>` (o slash command do plugin — definido em `commands/export-diagram.md` na raiz do repositório).
- O usuário pedir em linguagem natural para exportar, salvar, rasterizar, converter ou baixar um diagrama no formato `.svg` ou `.png`. Formulações típicas:
  - "exporte isto como PNG"
  - "salve como SVG"
  - "me dê um PNG desse diagrama"
  - "rasterize"
  - "converta para png e svg"

O slash command é um wrapper fino que delega para cá — ambos os caminhos executam o mesmo procedimento abaixo.

## Escopo

Ambos os formatos são **somente o diagrama** — apenas o nó `<svg>`. Envoltórios editoriais (cabeçalho, cards de resumo, rodapé nas variantes `-full`) são descartados intencionalmente: o entregável da exportação é o próprio diagrama, adequado para Figma, slides, cards de redes sociais ou imagens de blog.

A exportação somente SVG mantém o `<title>` e o `<desc>` da fonte junto com o diagrama. Seus IDs com prefixo por diagrama e por variante são o que tornam seguro incorporar inline vários SVGs exportados na mesma página, sem que uma figura resolva para o nome acessível de outra figura.

Se o usuário pedir explicitamente "uma captura de tela da página inteira, incluindo os cards", trata-se de um pedido diferente — recorra a uma captura de tela normal da página inteira pelo sistema operacional ou navegador do usuário.

## Procedimento de exportação SVG

1. Leia o arquivo HTML de origem.
2. Extraia o **primeiro** bloco `<svg ...>...</svg>`. Use uma regex multilinha ancorada em `<svg` e `</svg>`. A maioria dos diagramas gerados tem apenas um SVG; se houver vários, o primeiro é o diagrama (arquivos de galeria são uma exceção — veja *Casos de borda*).
3. Torne-o autônomo:
   - Garanta que a tag de abertura tenha `xmlns="http://www.w3.org/2000/svg"`. Adicione-o se estiver faltando.
   - Garanta que um `viewBox` esteja presente. Os templates da skill sempre incluem um; avise o usuário se estiver ausente em vez de adivinhar.
   - Preserve `role="img"`, `aria-labelledby` e o `<title>` / `<desc>` do primeiro filho exatamente como foram escritos.
   - Injete o bloco `@font-face` de Amazon Ember / Amazon Ember Mono (forma canônica em [`assets/fonts/README.md`](../assets/fonts/README.md)) em um `<style>` para que o SVG seja renderizado com a tipografia correta onde os arquivos de fonte estiverem acessíveis. Aponte as fontes `url()` para um caminho que resolva **a partir da própria localização do SVG** (ex.: `../../skills/aws-diagram-design/assets/fonts/woff2` para um SVG dentro deste repositório), e sempre mantenha as fontes `local()` primeiro — uma máquina com as fontes instaladas não precisa de nenhum acesso a arquivos. Se o SVG for circular sozinho, copie os arquivos woff2 para junto dele, remova as fontes `url()` e conte com `local()` + a pilha de fallback, ou recomende a exportação PNG. Nenhum CDN externo de fontes é referenciado.
     ```svg
     <defs>
       <style>
         @font-face { font-family: 'Amazon Ember'; font-weight: 400; src: local('Amazon Ember'), url('FONTS/AmazonEmber_W_Rg.woff2') format('woff2'); }
         /* …remaining faces from assets/fonts/README.md… */
       </style>
     </defs>
     ```
     Se o SVG já contiver um bloco `<defs>` (ou um `<style>` dentro dele), **mescle** nele — não adicione um segundo `<defs>`. Qualquer valor CSS que contenha `&` deve ser escapado em XML como `&amp;` — um `.svg` autônomo é analisado como XML estrito.
4. Adicione no início `<?xml version="1.0" encoding="UTF-8"?>\n` para que o arquivo seja um XML bem formado.
5. Grave em `<basename>.svg` ao lado da fonte (ex.: `example-architecture.html` → `example-architecture.svg`). Respeite um caminho de saída explícito se o usuário fornecer um.

### Ressalva a comunicar ao usuário

Ferramentas que não carregam fontes vinculadas no momento da importação (Illustrator offline, alguns caminhos de importação do Figma, visualizadores SVG mais antigos) vão substituir a tipografia. O SVG é renderizado corretamente em qualquer navegador moderno. Para portabilidade perfeita em pixels, recomende a exportação PNG.

## Procedimento de exportação PNG

Renderize **o HTML original** (não o SVG extraído) e capture a tela apenas da caixa delimitadora do elemento `<svg>`. Isso mantém o carregamento de fontes confiável (já configurado no HTML de origem) e, ao mesmo tempo, atende à regra de "somente o diagrama". O PNG sempre tem **fundo transparente** (`omit_background=True`), para que possa ser colocado sobre qualquer cor de slide ou documento sem um halo branco. Para HTML com movimento habilitado, acrescente `?motion=static`, aguarde `document.fonts.ready` e confirme que a raiz de movimento tem `data-frame="static"` antes da captura; nunca exporte após um atraso arbitrário de tempo de relógio.

### Detecção

Antes de executar qualquer coisa, verifique se o Playwright está instalado:

```
python -c "import playwright" 2>NUL || python -c "import playwright"
```

Se a importação falhar, apresente exatamente esta instrução ao usuário e pare:

> O Playwright não está instalado. Para habilitar a exportação PNG, execute:
> ```
> pip install playwright
> playwright install chromium
> ```
> Depois, peça para eu exportar novamente.

Não instale automaticamente. O usuário pediu um recurso, não uma mudança no sistema.

### Rasterizar

Grave o trecho abaixo em um arquivo temporário e execute-o com `python <tmp.py> <src.html> <out.png>`:

```python
from playwright.sync_api import sync_playwright
import sys, pathlib

src, out = sys.argv[1], sys.argv[2]
scale = int(sys.argv[3]) if len(sys.argv) > 3 else 2

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(device_scale_factor=scale)
    page.goto(f"file://{pathlib.Path(src).resolve()}")
    page.wait_for_load_state("networkidle")
    page.locator("svg").first.screenshot(path=out, omit_background=True)
    browser.close()
```

Padrão `device_scale_factor=2` para uma saída nítida. Aceite `1` para assets compactos ou `3` para impressão/uso hero em retina, passado como terceiro argumento de CLI.

### Nomenclatura da saída

`example-architecture.html` → `example-architecture.png`, gravado ao lado da fonte. Respeite caminhos explícitos fornecidos pelo usuário.

## Dimensionando a exportação (Sizing the export)

As dimensões em pixels do PNG são o `viewBox` do SVG × `device_scale_factor`. Portanto, a decisão de tamanho já foi tomada quando o diagrama foi desenhado — veja [`output-spec.md` §2](output-spec.md) para os presets. A exportação apenas escolhe o multiplicador.

| Destino | Escala | Resultado a partir de um `viewBox` 1280×720 |
|---|---|---|
| Documentação, README, wiki | 2 | 2560×1440 |
| Apresentação de slides (projetada) | 2 | 2560×1440 |
| Impressão / apostila em PDF | 3 | 3840×2160 |
| Miniatura inline, e-mail | 1 | 1280×720 |

### Atingindo um tamanho exato em pixels

Quando o usuário precisar de dimensões específicas (um card OG com exatamente 1200×630, uma imagem de slide em 1920×1080), calcule o fator de escala em vez de adivinhar — o Playwright aceita valores fracionários:

```
scale = target_width / viewBox_width
```

Um `viewBox` de 960 de largura para um alvo de 1200px é `scale=1.25`. Duas regras:

- **Nunca use escala abaixo de 1** para atingir um alvo pequeno — isso desfoca a tipografia. Redesenhe em um preset menor.
- **Nunca use escala acima de 4** — além disso você está ampliando um layout que foi projetado para uma tela menor; redesenhe em `slide-16x9` ou em um preset de impressão.

Se a proporção do alvo não corresponder à proporção do `viewBox`, diga isso e ofereça redesenhar no preset correspondente. Adicionar preenchimento ou recortar um diagrama finalizado para caber em um quadro não é uma operação de exportação — isso quebra a margem de segurança de 40px.

## Casos de borda

- **A fonte é `assets/index.html`** (a galeria, vários SVGs em um arquivo): recuse a exportação e pergunte ao usuário a qual arquivo de diagrama específico ele se referia. Não adivinhe.
- **Nenhum bloco `<svg>` encontrado**: a fonte não é um arquivo de diagrama. Informe o usuário; não grave nada.
- **O HTML ao redor importa para o usuário**: ele quer os cards/cabeçalho na imagem. Diga que esta skill exporta somente diagramas e recomende uma captura de tela da página inteira pelo navegador (ou uma impressão em PDF separada).
- **A fonte não tem as fontes tipográficas em tempo de execução**: o Playwright vai substituí-las e a captura ficará estranha. Verifique se o HTML de origem contém o bloco `@font-face` de Amazon Ember / Ember Mono (com caminhos `url()` resolvíveis) ou se as fontes estão instaladas no sistema operacional (`./install.sh fonts`). Se nenhuma das duas condições for verdadeira, corrija a fonte em vez de contornar o problema na exportação.

## O que este comando nunca faz

- Modificar o HTML de origem.
- Adicionar botões de exportação ou tags `<script>`. Diagramas estáticos permanecem sem scripts; uma fonte que já tenha movimento habilitado pode manter o controlador com escopo de [`animation.md`](animation.md), mas a exportação nunca injeta outro controlador.
- Emitir automaticamente `.svg` ou `.png` junto com a geração do HTML. Manual em toda chamada.
- Incorporar um envoltório HTML (cards, cabeçalhos) no SVG via `foreignObject`. Frágil demais entre renderizadores.
