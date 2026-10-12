---
inclusion: manual
---

# Exportar diagrama (aws-diagram-design)

Equivalente no Kiro ao comando `/aws-diagram-design:export-diagram` do Claude Code. Siga, como fonte da verdade, `references/export.md` dentro do diretório da skill (`.kiro/skills/aws-diagram-design/` ou `~/.kiro/skills/aws-diagram-design/`).

## Entrada

O usuário informa o arquivo `.html` do diagrama e, opcionalmente: só SVG, só PNG, escala do PNG (`1`, `2` padrão ou `3`) e caminho de saída. Padrão: gerar `.svg` **e** `.png` ao lado do HTML.

## Comportamento obrigatório

1. Sem arquivo informado → pergunte qual `.html`. Não adivinhe.
2. Origem `assets/index.html` (galeria com vários SVGs) → recuse e pergunte qual diagrama.
3. Sem bloco `<svg>` → recuse e não grave nada.
4. PNG pedido:
   - Se existir o ambiente virtual `.venv-diagramas/` na raiz do workspace, use `.venv-diagramas/bin/python` (Windows: `.venv-diagramas\Scripts\python.exe`) para verificar e executar o Playwright.
   - Se o Playwright não estiver instalado, mostre a instrução de instalação de `export.md` e pare. **Nunca instale automaticamente.**
5. Escala fora de {1, 2, 3} → rejeite.

Ao final, informe caminhos e tamanhos dos arquivos gerados.
