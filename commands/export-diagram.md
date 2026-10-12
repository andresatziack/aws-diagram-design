---
description: Exporta um arquivo de diagrama HTML do aws-diagram-design para .svg e .png ao lado do arquivo de origem
argument-hint: <html-file> [--svg-only|--png-only] [--scale=N] [--output=<path>]
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
---

Exporte o HTML do diagrama em `$1` para `.svg` e/ou `.png`, seguindo o procedimento documentado em [`skills/aws-diagram-design/references/export.md`](../skills/aws-diagram-design/references/export.md). Trate essa referência como a fonte da verdade — não reimplemente a lógica aqui.

String completa de argumentos: `$ARGUMENTS`

## Padrões

- Produza **ambos** `.svg` e `.png` ao lado do arquivo de origem (ex.: `diagram.html` → `diagram.svg` + `diagram.png`).
- O PNG é renderizado com `device_scale_factor=2`.

## Flags

- `--svg-only` — gera apenas o SVG. Ignora o Playwright completamente.
- `--png-only` — gera apenas o PNG.
- `--scale=1` / `--scale=2` / `--scale=3` — sobrescreve o fator de escala do dispositivo do PNG. Padrão `2`.
- `--output=<path>` — sobrescreve o caminho base de saída; a extensão do formato é acrescentada. Aplica-se a ambos os formatos quando ambos são produzidos.

## Comportamento obrigatório

1. **Nenhum caminho de origem fornecido** → pergunte ao usuário qual arquivo `.html` exportar. Não adivinhe.
2. **A origem é `assets/index.html`** (a galeria, vários SVGs em um único arquivo) → recuse e pergunte qual arquivo de diagrama específico. Conforme a seção de casos extremos da referência.
3. **A origem não tem bloco `<svg>`** → recuse e informe o usuário; não grave nada.
4. **PNG solicitado, mas Playwright não instalado** → apresente a instrução de instalação da referência literalmente e pare. **Não** instale automaticamente.
5. **PNG solicitado com `--scale` fora de {1,2,3}** → rejeite; os valores válidos são 1, 2, 3.

Depois de produzir as saídas, informe ao usuário os caminhos e tamanhos dos arquivos.
