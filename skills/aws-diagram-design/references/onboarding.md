# Onboarding — gere sua skin (tema) a partir de uma fonte de design

**Objetivo:** apontar a skill para uma fonte de design — um site, uma skill instalada ou uma pasta local — e fazê-la extrair a paleta + tipografia, depois reescrever `style-guide.md` para que todo diagrama futuro herde essa skin.

Leva cerca de 60 segundos.

Três métodos de origem são suportados. Vá para a seção relevante:

- [§ URL](#url) — buscar um site ao vivo
- [§ Skill](#skill) — ler uma Agent Skill instalada que contém design tokens
- [§ Folder](#folder) — ler um diretório local de design system (CSS, JSON, Markdown)

---

## O fluxo (todos os métodos)

```
Source you provide (URL / skill name / folder path)
      ↓
[1] read / fetch the source
      ↓
[2] extract dominant colors + fonts
      ↓
[3] map to semantic roles (paper, ink, muted, accent, …)
      ↓
[4] propose a style-guide.md diff
      ↓
[5] write the diff (with your approval)
      ↓
future diagrams use your tokens
```

---

---

## § URL

### Invocação

> *"Faça o onboarding do Schematic para o meu site — `https://example.com`"*

---

### Etapa 1 — buscar a página

Use `agent-browser` (preferido) ou um `fetch` simples. Se o site tiver várias páginas que valham a amostragem (landing + blog + produto), busque 2–3 e combine os sinais de paleta.

```bash
agent-browser navigate https://example.com --screenshot out.png --html out.html
```

---

## Etapa 2 — extrair cores e fontes

### Cores

Analise o CSS renderizado e a captura de tela:

- **Cor de fundo** do `<body>` ou da região grande dominante → `paper`
- **Cor primária do texto** (texto do corpo) → `ink`
- **Cor secundária do texto** (legendas, metadados) → `muted`
- **Cor de marca mais usada** (botão de CTA, link, destaque de título) → `accent`
- **Fundo de contêiner / card** um pouco mais escuro que o paper → `paper-2`
- **Cor de borda / linha fina (hairline)** → `rule` (converta para rgba de ink com opacidade ~0.12)

Prefira propriedades personalizadas de CSS quando o site as expuser (`:root { --accent: …; }`). Caso contrário, extraia por meio de amostras de `getComputedStyle` renderizado ou de uma passada de histograma de cores sobre a captura de tela.

### Fontes

Leia a pilha `font-family` renderizada de:

- `<h1>` → família de `title`
- `<body>` → família de `node-name`  
- `<code>`, `<pre>` ou qualquer elemento com estilo mono → família de `sublabel`

Se o site tiver apenas uma família, mantenha os padrões do schematic para os papéis ausentes (Amazon Ember para o título, Amazon Ember Mono para mono). Não force a escolha de uma fonte mono que não esteja no site.

### Portão de fonte exata para saída alinhada à marca

Não substitua uma família de marca detectada por `serif`, `system-ui` ou `ui-monospace` apenas para deixar o arquivo livre de dependências. Uma fonte pública faz parte do sistema visual.

1. Registre a família e o peso computados usados pelos elementos de título, corpo e rótulo técnico amostrados.
2. Rastreie cada família até sua origem: uma folha de estilo existente do Google Fonts, uma pilha instalada/do sistema ou um `@font-face` hospedado de forma personalizada.
3. Se estiver disponível pelo Google Fonts, leve o nome exato da família, os pesos e a folha de estilo aprovada para o guia de estilo e o HTML gerado. A allowlist de arquivo único aceita apenas uma URL HTTPS analisada cujo hostname seja exatamente `fonts.googleapis.com` e cujo caminho seja exatamente `/css2`; hosts com prefixo/semelhantes e outros caminhos falham. Preserve uma pilha de sistema intencional na ordem e verifique a família resolvida na máquina de destino.
4. Uma fonte hospedada de forma personalizada ou paga não é compatível com a allowlist padrão de arquivo único. Rotule esse papel como `fallback`, a menos que o usuário aprove e empacote a fonte separadamente; nunca adicione silenciosamente uma URL de fonte remota nem alegue correspondência exata.
5. Verifique a saída renderizada com `getComputedStyle`; uma família declarada que não carregou não passa.

Para uma página que contenha diagramas sob medida ou figuras editoriais, inspecione os papéis de fonte renderizados delas, além do artigo ao redor. Uma folha de estilo específica da figura pode diferir intencionalmente da pilha global de título/corpo do site.

---

## Etapa 3 — mapear para papéis semânticos

Proponha um diff preenchendo esta tabela:

| Papel | Detectado | Confiança |
|---|---|---|
| paper | `#f8f6f0` | alta |
| ink | `#111111` | alta |
| muted | `#6b6b68` | média |
| accent | `#c73a2b` | alta |
| … | … | … |

Sinalize palpites de baixa confiança para que o usuário possa corrigi-los antes de aplicar.

### Verificações de restrições

Antes de escrever, valide:

- **Contraste AA**: `ink` sobre `paper` ≥ 4.5:1. `muted` sobre `paper` ≥ 4.5:1 para texto do corpo.
- **Accent é a cor mais saturada**: não meio apagada, não próxima do cinza.
- **paper ≠ branco puro**: se o site usar `#ffffff`, recorra a `#fafaf7` para preservar a sensação neutra e quente do Schematic — ou peça ao usuário que confirme que o branco puro é intencional.

Se alguma verificação falhar, proponha um valor ajustado e explique o porquê.

---

## Etapa 4 — pré-visualizar o diff

Mostre ao usuário o que vai mudar em `style-guide.md`. Apenas a tabela de tokens — todo o resto permanece igual.

```diff
-| `paper`  | `#f5f4ed` | `#1c1a17` |
-| `ink`    | `#0b0d0b` | `#f1efe7` |
-| `accent` | `#f7591f` | `#ff6a30` |
+| `paper`  | `#f8f6f0` | `#1a1815` |
+| `ink`    | `#111111` | `#efeee7` |
+| `accent` | `#c73a2b` | `#e05440` |
```

Regenere também a variante escura pela regra de inversão (`rgba(11,13,11, X)` → `rgba(ink-rgb, X)`).

Inclua um **recibo de fidelidade à marca** compacto junto com a pré-visualização:

- URLs amostradas;
- valores detectados de paper, ink, muted, accent, superfície e rule;
- famílias de título, corpo e rótulo técnico com pesos e URLs de origem;
- `exact` ou `fallback` para cada papel de fonte;
- qualquer estilo de figura específico da página que deva sobrescrever a skin global do site.

O recibo é obrigatório quando o usuário diz “combine com este site”, “use a identidade visual deles” ou fornece uma página como referência visual.

---

## Etapa 5 — aplicar

Escreva os novos tokens em `style-guide.md`. Sugira executar o fluxo `/regenerate-examples` (se existir) ou reconstruir um exemplo para verificar se a nova skin fica legível.

Após o onboarding, o usuário deve:

1. Abrir `assets/index.html` (galeria) e confirmar que a nova paleta parece coerente em todos os 27 tipos.
2. Se algum tipo parecer estranho, geralmente é preciso ajustar `muted` (muitas vezes escuro ou claro demais em relação ao novo `paper`).

---

## Quando o onboarding por URL falha

- **O site usa webfonts que você não consegue replicar** (hospedadas de forma personalizada, pagas): mantenha os padrões do schematic para tipografia e aplique a skin apenas às cores.
- **A marca tem 6+ cores** e você não consegue identificar uma hierarquia clara: escolha uma como `accent`, rebaixe as demais para variantes de `muted` ou ignore-as. A gramática do schematic usa apenas 5–7 papéis.
- **O site é dark-mode first**: inverta a inversão — trate o paper escuro deles como o `paper` padrão e gere uma variante clara por inversão.
- **A página inicial é só imagens, sem texto**: peça uma URL de blog ou de documentação — páginas com muito texto expõem a hierarquia tipográfica.

---

## § Skill

Extraia tokens de uma Agent Skill instalada que traga seu próprio design system (por exemplo, uma skill `brand-design` ou `ui-kit`).

### Invocação

> *"Faça o onboarding do Schematic a partir da minha skill `acme-design`"*

Ou o portão oferece isto como opção (b) e o usuário informa o nome da skill.

### Etapa 1 — localizar a skill

Use o local de skills instaladas exposto pelo agente atual quando disponível. Caso contrário, procure nos locais do harness ativo:

**Pi:**

1. `~/.pi/agent/skills/<skill-name>/` e `~/.agents/skills/<skill-name>/` (instalações do usuário)
2. `.pi/skills/<skill-name>/` no diretório atual, além de `.agents/skills/<skill-name>/` do diretório atual até a raiz do repositório (instalações do projeto)
3. Caminhos de pacotes listados em `~/.pi/agent/settings.json` ou `.pi/settings.json`; pacotes gerenciados ficam em `~/.pi/agent/git/`, `~/.pi/agent/npm/`, `.pi/git/` ou `.pi/npm/`

**Claude Code:**

1. `~/.claude/skills/<skill-name>/` (instalação do usuário)
2. `.claude/skills/<skill-name>/` (instalação do projeto)

**Kiro:**

1. `.kiro/skills/<skill-name>/` na raiz do workspace (instalação do projeto — tem prioridade quando o nome se repete)
2. `~/.kiro/skills/<skill-name>/` (instalação do usuário; não disponível no Kiro Web)

Por fim, verifique qualquer caminho que o usuário fornecer explicitamente. Se a skill ainda não for encontrada, peça ao usuário que confirme o nome ou forneça o caminho.

### Etapa 2 — ler as fontes de tokens

Faça glob no diretório da skill buscando qualquer um destes arquivos e leia todos:

| Prioridade | Padrão | O que procurar |
|---|---|---|
| 1 | `*.css`, `colors*.css`, `tokens.css` | Propriedades personalizadas de CSS em `:root { --color-*: …; }` |
| 2 | `tokens.json`, `design-tokens.json`, `*.tokens.json` | JSON de tokens do Style Dictionary / Figma |
| 3 | `SKILL.md`, `README.md` | Tabelas Markdown listando cores, fontes, valores hex |
| 4 | `style-guide.md`, `*design*.md` | Qualquer documentação de design narrativa |
| 5 | `*.html` (arquivos de pré-visualização/exemplo) | Blocos `<style>` inline — examine as regras de `:root` e `body` |

Leia todas as correspondências e combine — propriedades personalizadas de CSS têm prioridade sobre valores inferidos do HTML.

### Etapa 3 — extrair cores e fontes

**De propriedades personalizadas de CSS:**
Mapeie nomes de variáveis para papéis semânticos usando heurísticas de nome:

| Se o nome da variável contiver… | Mapear para o papel |
|---|---|
| `background`, `bg`, `paper`, `surface`, `canvas` | `paper` |
| `foreground`, `text`, `body`, `ink`, `on-surface` | `ink` |
| `muted`, `subtle`, `secondary`, `caption` | `muted` |
| `accent`, `brand`, `primary`, `cta`, `highlight` | `accent` |
| `border`, `rule`, `divider`, `outline` | `rule` |
| `mono`, `code`, `pre` | fonte de `sublabel` |

**De tokens JSON:** siga as mesmas heurísticas nos nomes das chaves. Se o JSON seguir o formato Style Dictionary (`{ "color": { "brand": { "value": "#…" } } }`), achate o caminho e aplique as heurísticas à chave folha.

**De tabelas Markdown:** procure linhas com valores hex (`#rrggbb`) adjacentes a palavras com cara de papel. Uma linha como `| accent | #eb6c36 |` mapeia diretamente.

**Fontes:** procure regras `font-family`, declarações `@import` ou `@font-face` e menções em Markdown a nomes de fontes junto com tamanho/peso.

### Etapa 4 — mapear, validar, propor diff

Igual ao método por URL: preencha a tabela de papéis, execute as verificações de contraste, mostre o diff e peça aprovação antes de escrever.

### Quando a extração da skill é ambígua

- **A skill não tem arquivos CSS ou de tokens**: recorra à leitura de todos os arquivos `.md` e procure valores hex mencionados na prosa. Mostre o que encontrou e peça ao usuário que confirme os mapeamentos antes de aplicar.
- **Vários candidatos a accent**: liste-os e peça ao usuário que escolha um. Não adivinhe.
- **A skill é dark-mode first**: pergunte se os valores escuros devem ser tratados como os padrões de `paper`/`ink` ou se deve inverter.

---

## § Folder

Extraia tokens de um diretório local — um repositório de design system clonado, uma exportação do Figma ou qualquer pasta para a qual o usuário apontar.

### Invocação

> *"Faça o onboarding do Schematic a partir do meu design system em `~/projects/brand/design-tokens/`"*

Ou o portão oferece isto como opção (c) e o usuário fornece o caminho.

### Etapa 1 — descobrir arquivos

Faça glob na pasta (recursivamente, até 3 níveis de profundidade) buscando:

```
**/*.css
**/*.scss        (read @forward / $variable declarations)
**/tokens.json
**/*.tokens.json
**/design-tokens.json
**/colors.json
**/*style-guide*.md
**/*design-system*.md
**/README.md
**/*.html        (scan <style> blocks only)
```

Se o conjunto de resultados for grande (>20 arquivos), prefira arquivos na raiz e arquivos cujos nomes contenham `color`, `token`, `brand`, `palette`, `style` ou `theme`.

### Etapa 2 — ler e combinar

Leia todos os arquivos descobertos. Aplique a mesma lógica de extração do método Skill (§ Skill → Etapa 3). Propriedades personalizadas de CSS e tokens JSON têm prioridade sobre valores inferidos da prosa.

**Variáveis SCSS:** trate `$variable-name: value;` da mesma forma que uma propriedade personalizada de CSS — aplique as heurísticas de nome a `$variable-name`.

**JSON de tokens do Figma** (formato do Figma Tokens Plugin):

```json
{ "colors": { "brand": { "primary": { "value": "#eb6c36", "type": "color" } } } }
```

Percorra a árvore; os campos `value` das folhas são as cores, e os segmentos do caminho fornecem a heurística de papel.

### Etapa 3 — mapear, validar, propor diff

Igual ao método por URL: execute as verificações de contraste, mostre o diff completo em relação ao `style-guide.md` atual e escreva somente depois que o usuário aprovar.

### Quando a extração da pasta é ambígua

- **Nenhum arquivo de tokens estruturado, apenas documentação em prosa**: leia todos os `.md` na raiz e extraia os valores hex encontrados perto de palavras com cara de papel. Mostre ao usuário uma tabela do que você inferiu — não aplique silenciosamente mapeamentos incertos.
- **Vários temas / esquemas de cores encontrados**: liste-os e pergunte ao usuário qual usar como skin do diagrama.
- **A pasta não tem nenhum arquivo legível**: avise o usuário e peça um caminho mais específico ou mude para a entrada manual de tokens.

---

## Futuro: skins por projeto

Se o usuário quiser várias skins (uma por projeto), duplique `style-guide.md` como `style-guides/<project>.md` e adicione um comentário de cabeçalho apontando o build para a skin ativa. Esse é um recurso da v5.2 — por enquanto, uma skin por instalação da skill.
