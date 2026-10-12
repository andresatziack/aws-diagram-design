# Amazon Ember — fontes incluídas

Amazon Ember (e Amazon Ember Mono) conforme distribuídas pela Amazon em
<https://developer.amazon.com/en-US/alexa/branding/echo-guidelines/identity-guidelines/typography>
(`Amazon_Typefaces_Complete_Font_Set_Mar2020.zip`). Termos de uso:
[`Amazon-Ember-Licensing-Guidelines.pdf`](Amazon-Ember-Licensing-Guidelines.pdf) neste diretório.

## Inventário

| Arquivo (woff2/) | Família | Peso / estilo |
|---|---|---|
| `AmazonEmber_W_Lt.woff2` | Amazon Ember | 300 |
| `AmazonEmber_W_Rg.woff2` | Amazon Ember | 400 |
| `AmazonEmber_W_RgIt.woff2` | Amazon Ember | 400 itálico |
| `AmazonEmber_W_SBd.woff2` | Amazon Ember | 600 |
| `AmazonEmber_W_Bd.woff2` | Amazon Ember | 700 |
| `AmazonEmber_W_He.woff2` | Amazon Ember | 800 |
| `AmazonEmberMono_W_Rg.woff2` | Amazon Ember Mono | 400 |
| `AmazonEmberMono_W_Bd.woff2` | Amazon Ember Mono | 700 |

`ttf/` contém as fontes para desktop (Ember Lt/Rg/RgIt/Medium/Bd/He + Ember Mono Rg/Bd) para instalação no sistema operacional via
`./install.sh fonts` (raiz do repositório).

## Bloco @font-face canônico

Cole no HTML do diagrama gerado, ajustando `FONTS` para o caminho que chega a este
diretório a partir do arquivo de saída (veja `references/style-guide.md § Pilha de fontes (Font stack)` para as
regras de ordem de carregamento):

```css
/* FONTS = relative or absolute path to <skill-dir>/assets/fonts/woff2 */
@font-face { font-family: 'Amazon Ember'; font-weight: 300; font-style: normal;
  src: local('Amazon Ember Light'), local('Amazon Ember'), url('FONTS/AmazonEmber_W_Lt.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember'; font-weight: 400; font-style: normal;
  src: local('Amazon Ember'), url('FONTS/AmazonEmber_W_Rg.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember'; font-weight: 400; font-style: italic;
  src: local('Amazon Ember Italic'), local('Amazon Ember'), url('FONTS/AmazonEmber_W_RgIt.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember'; font-weight: 600; font-style: normal;
  src: local('Amazon Ember Medium'), local('Amazon Ember'), url('FONTS/AmazonEmber_W_SBd.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember'; font-weight: 700; font-style: normal;
  src: local('Amazon Ember Bold'), local('Amazon Ember'), url('FONTS/AmazonEmber_W_Bd.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember'; font-weight: 800; font-style: normal;
  src: local('Amazon Ember Heavy'), local('Amazon Ember'), url('FONTS/AmazonEmber_W_He.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember Mono'; font-weight: 400; font-style: normal;
  src: local('Amazon Ember Mono'), url('FONTS/AmazonEmberMono_W_Rg.woff2') format('woff2'); }
@font-face { font-family: 'Amazon Ember Mono'; font-weight: 700; font-style: normal;
  src: local('Amazon Ember Mono Bold'), local('Amazon Ember Mono'), url('FONTS/AmazonEmberMono_W_Bd.woff2') format('woff2'); }
```

`local()` primeiro — uma máquina com as fontes instaladas nunca baixa os arquivos. Sempre mantenha as
pilhas de fallback nas declarações `font-family` (`'Helvetica Neue', Helvetica, Arial, sans-serif`
para Ember; `ui-monospace, monospace` para Ember Mono); ambientes que bloqueiam fontes de arquivo
(ex.: o sandbox de SVG do GitHub) recorrem às fontes do sistema. Ember Mono vem com 400/700 — especificações mono
em 500/600 se ajustam ao peso mais próximo. Nenhuma CDN externa de fontes é necessária em lugar algum.
