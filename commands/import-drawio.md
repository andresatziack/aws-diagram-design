---
description: Redesenha um arquivo draw.io como um diagrama editorial no formato, tamanho e nível de detalhe escolhidos
argument-hint: <drawio-file> [--format=html|svg|png|html+png] [--size=<preset>] [--detail=faithful|balanced|simplified] [--audience=engineer|mixed|executive] [--type=<diagram-type>] [--page=N|NAME|all] [--variant=light|dark|full] [--output=<path>]
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
---

Redesenhe o arquivo draw.io em `$1` como um diagrama no design system desta skill, seguindo [`skills/aws-diagram-design/references/import-drawio.md`](../skills/aws-diagram-design/references/import-drawio.md) e [`skills/aws-diagram-design/references/output-spec.md`](../skills/aws-diagram-design/references/output-spec.md). Trate essas referências como a fonte da verdade — não reimplemente a lógica aqui.

String completa de argumentos: `$ARGUMENTS`

Aceita `.drawio`, `.drawio.xml`, `.xml`, `.drawio.png` e `.drawio.svg`.

## Padrões

- `--format=html` — um arquivo HTML autocontido ao lado do arquivo de origem.
- `--size=doc-inline` — `viewBox 0 0 960 600`.
- `--detail=balanced` · `--audience=mixed`.
- `--variant=light` — o template claro mínimo.
- Um arquivo de página única seleciona sua única página; um arquivo com várias páginas lista as páginas e pergunta qual usar.
- O tipo é escolhido a partir da estrutura extraída; `--type` força um dos 27.

## Flags

- `--format` — `html` (padrão), `svg`, `png` ou `html+png`. Formatos não HTML são produzidos a partir do HTML via `references/export.md`, nunca escritos à mão.
- `--size` — qualquer preset em `output-spec.md` §2: `doc-inline`, `doc-wide`, `slide-16x9`, `slide-4x3`, `social-og`, `social-square`, `print-a4-landscape`, `print-letter-landscape`, `fit`.
- `--detail` — `faithful` (≤24 nós, em zonas), `balanced` (≤12), `simplified` (≤7).
- `--audience` — `engineer`, `mixed`, `executive`. Governa a redação, não a quantidade de elementos.
- `--type` — força um tipo de diagrama em vez de inferi-lo.
- `--page` — índice da página, nome da página ou `all` (um arquivo por página).
- `--variant` — template editorial `light`, `dark` ou `full`.
- `--output` — caminho base de saída; a extensão é acrescentada conforme o formato.

## Comportamento obrigatório

1. **Nenhum arquivo fornecido** → pergunte qual arquivo `.drawio`. Não adivinhe.
2. **Sempre localize a skill instalada e execute `<skill-dir>/scripts/drawio_extract.py` primeiro.** Nunca presuma que a skill está sob o diretório de trabalho atual, e nunca leia um arquivo `.drawio` diretamente — a maioria é comprimida, e o XML bruto é ruído.
3. **O extrator sai com código diferente de zero** → informe a mensagem dele literalmente e pare.
4. **O digest mostra 0 nós** → a origem é apenas imagem ou está criptografada. Diga isso; peça o arquivo original. Não invente conteúdo.
5. **Arquivo com várias páginas sem `--page`** → liste as páginas com suas contagens de nós/arestas e pergunte qual usar.
6. **O nível de detalhe solicitado é impossível no tamanho solicitado** (ex.: `faithful` em uma origem de 40 nós em `slide-16x9`) → diga isso antes de desenhar e proponha visão geral + detalhe por zona.
7. **`--detail=faithful` acima de 9 nós** → divida o layout em zonas; acima de 24 nós, divida em arquivos de visão geral + detalhe.
8. **Nunca transporte coordenadas, cores ou fontes da origem.** A saída é um redesenho na skin (tema) do `style-guide.md` do projeto.
9. Execute o *taste gate* (filtro de gosto) do SKILL.md §9 e o checklist do `output-spec.md` §6 antes de gravar.

Depois de gravar os arquivos, informe os caminhos, tamanhos, os quatro controles (dials) usados e o *fidelity ledger* (registro de fidelidade) (o que foi mesclado, recolhido ou descartado).
