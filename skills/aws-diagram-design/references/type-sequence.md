# Sequência (Sequence)

**Melhor para:** fluxos de requisição/resposta, trocas de protocolo, interações entre vários atores ao longo do tempo, traces de chamadas de API, reconstruções de incidentes, caminhos de autenticação/renovação de token com ramificações.

## Convenções de layout
- Atores como caixas em uma linha horizontal no topo.
- **Linhas de vida (lifelines)**: linhas verticais tracejadas descendo de cada ator até a base.
- Mensagens: setas horizontais entre as linhas de vida; o tempo flui de cima→para baixo.
- **Barra de ativação**: retângulo estreito (`w=8`, preenchimento muted, traço de linha fina (hairline) de 0.8) sobre uma linha de vida, abrangendo o intervalo em que aquele ator detém o controle. Empilhe para chamadas aninhadas.
- Automensagens: loop curto em forma de U retornando à mesma linha de vida; rótulo à direita do loop.
- Mensagens de retorno: traço **tracejado** + marcador **preenchido** (nunca aberto). Prefira muted; opcionalmente use a mesma cor da chamada de origem ao parear pilhas de vários saltos. O sucesso principal pode usar coral sólido (veja Tipos de mensagem).
- Coral na resposta principal de sucesso ou na mensagem de destaque — uma, talvez duas. Traços focais de atores não contam para o orçamento de mensagens em coral.
- Quando o fluxo **se ramifica** (token válido vs inválido, retentativa, etapa opcional), desenhe uma moldura de **fragmento combinado** — não invente aglomerados soltos de setas if/else.

## Tipos de mensagem

| Tipo | Traço | Marcador | Quando |
|---|---|---|---|
| Chamada (síncrona) | muted ou link-blue sólido | preenchido | Requisição que espera uma resposta |
| Retorno | muted **tracejado** (ou mesma cor da chamada) | preenchido | Resposta a uma chamada síncrona — nunca sólido |
| Assíncrona / fire-and-forget | muted tracejado | ponta de seta **aberta** | Beacons, eventos, notificação unidirecional |
| Sucesso principal | accent sólido (≤1–2 mensagens) | accent preenchido | Somente a resposta principal do caminho feliz |

### Ponta de seta aberta (assíncrona)

Defina uma vez em `<defs>` e use somente para fire-and-forget:

```svg
<marker id="arrow-open" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
  <polyline points="0 0, 8 3, 0 6" fill="none" stroke="#545B64" stroke-width="1.2"/>
</marker>
```

Modo escuro: traço `#B6BEC9` (muted sobre paper escuro). Não preencha o marcador aberto — a ponta vazada é o sinal de assíncrono. Mensagens de retorno mantêm o marcador **preenchido** mesmo quando tracejadas.

## Fragmentos combinados (`alt` / `opt` / `loop`)

Use uma **moldura** retangular que abranja apenas as linhas de vida que participam da ramificação. O rótulo do operador é em Amazon Ember Mono, maiúsculas, em uma pequena aba no canto superior esquerdo da moldura. O tempo continua fluindo de cima→para baixo dentro da moldura.

### Primitiva de moldura (compartilhada)

```svg
<!-- Frame: light ink wash + hairline. Label tab top-left. -->
<rect x="X" y="Y" width="W" height="H" rx="4"
      fill="rgba(35,47,62,0.02)" stroke="rgba(35,47,62,0.22)" stroke-width="1"/>
<!-- Operator tab -->
<rect x="X" y="Y" width="40" height="16" rx="2"
      fill="#ffffff" stroke="rgba(35,47,62,0.22)" stroke-width="1"/>
<text x="X+20" y="Y+12" fill="#545B64" font-size="8"
      font-family="'Amazon Ember Mono', ui-monospace, monospace" text-anchor="middle"
      letter-spacing="0.12em">ALT</text>
```

Modo escuro: preenchimento da moldura `rgba(255,255,255,0.04)`, traço `rgba(255,255,255,0.22)`, preenchimento da aba = `paper` escuro (`#161E2D`), texto da aba = `muted` escuro (`#B6BEC9`).

### Operadores

| Operador | Regiões | Divisor | Rótulo de guarda |
|---|---|---|---|
| `opt` | 1 | nenhum | `[if condition]` sob a aba (Amazon Ember Mono 8px) |
| `alt` | **2 no máx.** | linha fina (hairline) horizontal tracejada atravessando a moldura | `[guard]` na região 1; `[else]` (ou uma segunda guarda) na região 2 |
| `loop` | 1 | nenhum | `[for each item]` ou `[retry ≤ 3]` sob a aba |

### Primitivas de guarda + divisor

```svg
<!-- Guard: left-aligned inside the frame, mono -->
<text x="X+12" y="GUARD_Y" fill="#545B64" font-size="8"
      font-family="'Amazon Ember Mono', ui-monospace, monospace" letter-spacing="0.04em">[token valid]</text>

<!-- alt region divider -->
<line x1="X+8" y1="DIV_Y" x2="X+W-8" y2="DIV_Y"
      stroke="rgba(35,47,62,0.20)" stroke-width="1" stroke-dasharray="4,3"/>
```

### Regras de layout de fragmentos
- Recuo esquerdo/direito da moldura ≥12px a partir dos centros das linhas de vida participantes mais externas (para que as barras de ativação fiquem dentro da moldura).
- ≥24px entre níveis y de mensagens consecutivas dentro de uma região (grade de 4px).
- A guarda fica nos primeiros ~20px sob a aba; a primeira mensagem daquela região fica ≥24px abaixo da linha de base da guarda.
- O y do divisor fica na grade de 4px; ≥16px livre das mensagens acima e abaixo.
- Fragmentos aninhados: **no máx. 1 nível**. Prefira dois diagramas separados a um aninhamento profundo.
- Padrão: **um** fragmento por diagrama. Um segundo somente se ambos ficarem abaixo do orçamento de complexidade.
- O coral fica em **uma** única mensagem de sucesso principal em todo o diagrama (geralmente o retorno do caminho feliz dentro da primeira região `alt`, ou o sucesso final fora de um loop). Não use coral nos dois ramos do `alt`.

### Fora do escopo (não invente)
- `par`, `critical`, `break`, `ref` e outros operadores UML — um segundo PR, se necessário.
- Criação/destruição de participantes, mensagens found/lost, barras de duração.

## Orçamento (limite) de complexidade (específico de sequência)
- Máx. de linhas de vida: 5 (igual à SKILL.md §7).
- Máx. de mensagens (setas): 12.
- Máx. de fragmentos combinados: 1 (padrão rígido); 2 somente se cada um for um `opt`/`loop` de região única.
- Máx. de regiões `alt`: 2.
- Profundidade máx. de aninhamento de fragmentos: 1.
- Máx. de elementos em coral: 2 (prefira 1 em diagramas com fragmentos).

Se exceder, divida: visão geral (caminho feliz) + detalhe (caminho de falha / renovação).

## Primitiva de linha de vida
```svg
<line x1="CX" y1="TOP" x2="CX" y2="BOTTOM"
      stroke="rgba(35,47,62,0.20)" stroke-width="1" stroke-dasharray="3,3"/>
```

## Primitiva de barra de ativação
```svg
<rect x="CX-4" y="TOP" width="8" height="H"
      fill="rgba(35,47,62,0.06)" stroke="#545B64" stroke-width="0.8"/>
```

## Antipadrões
- Seta de mensagem apontando *para cima* (inverte o tempo — nunca).
- Barras de ativação que nunca se fecham.
- Rótulos sobre outra linha de vida — encurte ou desloque o y para um vão.
- Raias no estilo swimlane em vez de linhas de vida (gramática diferente).
- Desenhar `if/else` como dois aglomerados soltos de setas **sem** moldura de fragmento.
- `alt` aninhado dentro de `alt` (divida em dois diagramas).
- Rótulo do operador do fragmento em Amazon Ember — deve ser mono: `ALT` / `OPT` / `LOOP`.
- Coral nos dois ramos do `alt`.
- Moldura que cobre atores sem mensagens dentro do fragmento.
- Ponta de seta preenchida em fire-and-forget assíncrono (use o marcador aberto).
- Ponta de seta aberta em mensagens de retorno (retornos permanecem preenchidos + tracejados).

## Exemplos
- `assets/example-sequence.html` — minimal claro (caminho feliz com cache frio)
- `assets/example-sequence-dark.html` — minimal escuro
- `assets/example-sequence-full.html` — full editorial
- `assets/example-sequence-oauth.html` — especial: chamada bearer + refresh em `alt` (claro)
- `assets/example-sequence-oauth-dark.html` — mesmo especial, escuro
- `assets/example-sequence-oauth-full.html` — mesmo especial, full editorial
