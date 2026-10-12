# Árvore / Hierarquia (Tree / Hierarchy)

**Ideal para:** organogramas, árvores de dependência, taxonomia, árvores de arquivos, decomposição de decisões, árvores de habilidades.

## Convenções de layout
- Raiz no topo, filhos distribuídos (fan-out) abaixo (ou raiz à esquerda, filhos à direita).
- Os nós são pequenos retângulos rotulados (`rx=6`), nome em Amazon Ember 12px 600 + sublabel opcional em Amazon Ember Mono 9px. Largura 120–180px, altura 40–52px.
- **Os conectores são ortogonais (estilo cotovelo), nunca diagonais.** O pai desce uma linha vertical curta, depois um barramento horizontal conecta os irmãos, e então cada filho tem uma curta queda vertical até sua borda superior. Traço muted de 1px.
- Indicador de folha: traço mais fino (0.8) ou preenchimento diferente — OU deixe a posição terminal fazer esse papel.
- Profundidade máxima: 4 (raiz + 3 níveis). Largura máxima por nível: 5.
- Coral em **um** nó: a raiz OU uma folha crítica. Não ambos.
- Desenhe os conectores antes dos nós.

## Antipadrões
- Árvore com 5+ níveis de profundidade em uma única página (ilegível — divida).
- Nós com larguras muito variadas — escolha no máximo 2 larguras.
- Linhas de conector diagonais.
- Níveis pulados (pai conectado ao neto sem o nível do meio).
- Coral na raiz E em uma folha.

## Exemplos
- `assets/example-tree.html` — minimal claro
- `assets/example-tree-dark.html` — minimal escuro
- `assets/example-tree-full.html` — editorial completo
