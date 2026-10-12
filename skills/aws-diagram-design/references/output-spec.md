# Especificação de saída da importação de Draw.io — formato × tamanho × detalhe × público

Quatro controles decidem o que um diagrama importado se torna. Defina-os **antes** de redesenhar — eles mudam o entregável, o layout, a escala tipográfica, a quantidade de nós e o texto, então ajustá-los depois significa redesenhar.

| Controle | Pergunta que responde | Padrão |
|---|---|---|
| **Formato** | Onde este arquivo vai parar? | `html` |
| **Tamanho** | Qual o tamanho da tela (canvas) e a que distância está o leitor? | `doc-inline` |
| **Nível de detalhe** | Reproduzir cada elemento ou comprimir? | `balanced` |
| **Público** | Quão técnico deve ser o texto? | `mixed` |

Infira as escolhas que estiverem claras no pedido (por exemplo, "para minha apresentação" implica um preset de slide). Faça uma pergunta concisa sobre qualquer ponto relevante que continue ambíguo. Se o usuário não se importar, use os padrões acima e diga quais usou.

---

## 1. Formato

| Formato | Entregável | Mantém | Descarta |
|---|---|---|---|
| `html` | `.html` autocontido (padrão) | cabeçalho, diagrama, cards de resumo, rodapé, fontes ao vivo | nada |
| `svg` | `.svg` ao lado da origem | o nó `<svg>`, texto vetorial | o invólucro editorial; as fontes são substituídas em ferramentas offline |
| `png` | `.png` em `device_scale_factor` | os pixels exatamente como o navegador os renderiza | a editabilidade vetorial |
| `html+png` | ambos | — | — |

Sempre gere o HTML primeiro — `svg` e `png` são produzidos *a partir* dele via [`export.md`](export.md). Nunca escreva um arquivo SVG à mão diretamente; o HTML é a fonte da verdade e o único artefato contra o qual o *taste gate* (portão de gosto, SKILL.md §9) foi escrito.

Escolha pelo destino:

| Destino | Formato | Preset de tamanho |
|---|---|---|
| Post de blog, README, site de documentação | `html` (incorporado) ou `png` | `doc-inline` |
| Keynote / PowerPoint / Google Slides | `png` @2 | `slide-16x9` |
| Figma / Illustrator / edição posterior | `svg` | `fit` |
| X / LinkedIn / card de link OG | `png` @2 | `social-og` |
| Material impresso, apresentação em PDF | `png` @3 | `print-a4-landscape` |
| Confluence / Notion / wiki interna | `png` @2 | `doc-wide` |

---

## 2. Tamanho

O preset define o `viewBox` do SVG. Todos os valores abaixo são divisíveis por 4, então a regra de grade do SKILL.md §7 continua valendo.

| Preset | viewBox | Proporção | PNG @2 | Escala tipográfica | Uso |
|---|---|---|---|---|---|
| `doc-inline` (padrão) | `0 0 960 600` | 8:5 | 1920×1200 | standard | Diagrama na largura do corpo de um post ou README |
| `doc-wide` | `0 0 1280 720` | 16:9 | 2560×1440 | standard | Documentação em largura total, páginas de wiki |
| `slide-16x9` | `0 0 1280 720` | 16:9 | 2560×1440 | presentation | Slide de apresentação, projetado |
| `slide-4x3` | `0 0 1024 768` | 4:3 | 2048×1536 | presentation | Templates de apresentação legados |
| `social-og` | `0 0 1200 632` | ~1.9:1 | 2400×1264 | presentation | Card de pré-visualização de link |
| `social-square` | `0 0 1080 1080` | 1:1 | 2160×2160 | presentation | Post de feed, carrossel |
| `print-a4-landscape` | `0 0 1120 792` | ~1.41:1 | @3 → 3360×2376 | print | A4 paisagem, margens de ~10mm a 96dpi |
| `print-letter-landscape` | `0 0 1056 816` | ~1.29:1 | @3 → 3168×2448 | print | US Letter paisagem |
| `fit` | derivado do conteúdo | qualquer | @2 | standard | Entrega vetorial; sem moldura fixa |

### Derivando `fit`

Arredonde a caixa delimitadora do conteúdo **para cima** até o próximo múltiplo de 4 e então adicione a moldura fixa: margem externa de 40px em todos os lados, mais 60px embaixo para a faixa da legenda. Nunca deixe o conteúdo encostar na borda do viewBox.

### Escala tipográfica por classe de tamanho

Os nomes dos nós encolhem em relação à tela à medida que ela cresce — resista a isso. Escale a tipografia com o preset para que um slide projetado continue legível da última fileira.

| Papel | standard | presentation | print |
|---|---|---|---|
| Título (Amazon Ember 700) | 28 | 40 | 32 |
| Nome do nó (Amazon Ember 600) | 12 | 16 | 12 |
| Sublabel (Amazon Ember Mono) | 9 | 12 | 9 |
| Rótulo de seta (Amazon Ember Mono) | 8 | 12 | 8 |
| Eyebrow / tag (Amazon Ember Mono) | 8 | 8 | 8 |
| Altura mínima da caixa do nó | 48 | 64 | 48 |
| Espaço mínimo entre nós | 24 | 40 | 24 |

A escala de apresentação implica menos nós — nomes de 16px em caixas de 64px consomem a tela. Se um layout `slide-16x9` não couber, é o controle de tamanho avisando que o controle de detalhe está alto demais; desça um nível em vez de diminuir a tipografia.

### Áreas seguras

- **Todos os presets:** margem externa de 40px; a faixa da legenda ocupa os 60px inferiores e nada mais fica ali.
- **`social-og`:** mantenha livres os 64px externos em todos os lados — os cortes dos cards de link são imprevisíveis entre plataformas.
- **`slide-*`:** mantenha livres os 80px inferiores se o template da apresentação tiver uma barra de rodapé; pergunte se não tiver certeza.

---

## 3. Nível de detalhe

Quanto da origem sobrevive. Este é um controle de *quantidade* — ele governa quantos elementos passam, não como são redigidos (isso é o §4).

| Nível | Nós | Arestas | Sublabels | O que sobrevive |
|---|---|---|---|---|
| `faithful` (詳細) | ≤24, em zonas | ≤32 | cada porta, protocolo, versão | Cada componente distinto da origem. Apenas duplicatas exatas são mescladas. |
| `balanced` (padrão) | ≤12 | ≤16 | sublabel técnico em ≤4 nós | Componentes que carregam a história; clusters de folhas colapsam em um nó cada. |
| `simplified` (簡略) | ≤7 | ≤9 | nenhum | Capacidades e sua sequência. A infraestrutura desaparece. |

`balanced` e `simplified` ficam dentro do orçamento (limite) de complexidade padrão (SKILL.md §7). **`faithful` o excede deliberadamente** — essa é a troca, e ela vem com condições:

1. **Zonas são obrigatórias.** Acima de 9 nós, todo nó pertence a uma zona rotulada (2–4 zonas, borda de linha fina (hairline), preenchimento `paper-2`, rótulo de zona mono em maiúsculas no canto superior esquerdo). Um diagrama de 20 nós sem zonas é um diagrama de fiação, não um schematic.
2. **As regras de conectores não relaxam.** As regras 1–5 do SKILL.md §6 continuam valendo com 24 nós. Se você não consegue rotear sem sobreposições, está acima do teto real — divida.
3. **Acima de 24 nós, divida.** Produza uma visão geral (zonas como nós, gramática `balanced`) mais um diagrama de detalhe por zona. Nomeie-os `<base>-overview.html`, `<base>-<zone>.html`. Nunca entregue uma tela única de 40 nós.
4. **O accent fica em 2.** Mais nós nunca compram mais elementos focais.

### Escada de degradação

Quando a origem tem mais do que o nível permite, corte nesta ordem e pare assim que estiver dentro do orçamento. Nunca corte de forma improvisada.

1. **Células decorativas** — notas adesivas, texto solto, blocos de título, marcas d'água, a própria legenda da origem. (Notas que valham a pena manter viram callouts de anotação — no máximo 2, veja [primitive-annotation.md](primitive-annotation.md).)
2. **Duplicatas exatas** — N workers/réplicas/shards idênticos viram um nó rotulado `Worker ×N`.
3. **Clusters de folhas** — um contêiner cujos filhos são todos folhas colapsa no contêiner: `Core Services` substitui suas três caixas. O extrator lista esses casos em *collapsible groups*.
4. **Sinks de grau 1 que não mudam a história** — um hook de monitoramento, um bucket de logs, uma camada de arquivamento.
5. **Infraestrutura transversal** — logging, métricas, segredos, CI. Em `simplified` saem sem perguntar; em `balanced` mantenha no máximo um, e só se o diagrama for sobre ele.
6. **Ainda acima?** Divida em visão geral + detalhe. Dividir é melhor que encolher.

Tudo o que for cortado nas etapas 2–6 vai para o *fidelity ledger* (registro de fidelidade, §5). A etapa 1 não precisa ser relatada.

---

## 4. Nível de público

Independente do controle de detalhe: os mesmos 12 nós recebem nomes diferentes para um time de plataforma e para um comitê diretivo. O detalhe define *quantos*; o público define *como se chamam*.

| Público | Nomes dos nós | Sublabels | Rótulos de arestas | Nunca |
|---|---|---|---|---|
| `engineer` | nomes exatos de serviço / componente | protocolo, porta, versão, tag de imagem | `POST /v2/orders`, `SQL`, `gRPC` | Verbos vagos como "conecta-se a" |
| `mixed` (padrão) | nomes de componentes, siglas expandidas | tecnologia apenas onde ela muda uma decisão | verbos simples — `verifies`, `writes`, `notifies` | Portas, versões, codinomes internos |
| `executive` | capacidades e resultados | nenhum | verbos de negócio — `approves`, `pays out` | Nomes de fornecedores, infraestrutura, protocolos |

Exemplo prático — o mesmo nó nos três níveis:

| Público | Nome do nó | Sublabel |
|---|---|---|
| `engineer` | `Auth Service` | `JWT · RS256 · :8443` |
| `mixed` | `Auth Service` | `token check` |
| `executive` | `Sign-in` | — |

Duas regras que valem em todos os níveis de público:

- **Nunca invente detalhes para preencher um espaço.** Se a origem diz `svc-04`, a saída `executive` diz o que ele faz somente se for possível deduzir pelo contexto — caso contrário pergunte, não adivinhe um nome de negócio.
- **Mantenha o vocabulário da origem para nomes próprios.** Renomear `Kafka` para `Message Bus` tudo bem em `executive`; renomeá-lo para `Event Grid` (um produto diferente) é um erro factual.

### Rótulos não latinos

Amazon Ember não tem cobertura CJK. Quando os rótulos contêm texto em japonês, chinês ou coreano, estenda a família nesses elementos `<text>` — não troque a skin inteira:

```svg
<text font-family="'Amazon Ember', 'Hiragino Sans', 'Noto Sans JP', 'Yu Gothic', sans-serif">認証サービス</text>
```

Para sublabels mono use `'Amazon Ember Mono', 'Noto Sans Mono CJK JP', monospace`. Glifos CJK são renderizados ~10% mais largos que os latinos no mesmo tamanho — dimensione a largura das caixas de acordo, e prefira nomes de 12px a sublabels de 8px para CJK, que fica borrado abaixo de 10px.

---

## 5. Fidelity ledger

Sempre que a saída for menor que a entrada — toda execução `balanced` e `simplified`, e a maioria das `faithful` — relate o que você cortou, no chat, depois do caminho do arquivo. Curto e específico:

```
Detail: balanced · 18 source nodes → 9 drawn
Merged:  worker-01..06 → "Ingest Worker ×6"
Collapsed: "Observability" group (Grafana, Loki, Tempo) → one node
Dropped: 2 sticky notes, CI pipeline (cross-cutting)
Kept in full: the request path (Client → Gateway → Orders → Postgres)
```

Quem lê o diagrama não consegue ver o que está faltando. Quem o pediu precisa saber.

---

## 6. Checklist

Execute junto com o *taste gate* do SKILL.md §9.

- [ ] Os quatro controles foram definidos — pedidos explicitamente, inferidos do destino ou assumidos como padrão e informados?
- [ ] O `viewBox` corresponde exatamente ao preset de tamanho, com valores divisíveis por 4?
- [ ] A escala tipográfica corresponde à classe de tamanho — e não a escala standard em um slide?
- [ ] Margem externa de 40px respeitada (64px para `social-og`)?
- [ ] Quantidade de nós dentro do teto do nível de detalhe?
- [ ] `faithful` acima de 9 nós → em zonas, e dividido acima de 24?
- [ ] Nomes dos nós, sublabels e rótulos de arestas todos no mesmo nível de público?
- [ ] Rótulos CJK com fallback de fonte?
- [ ] *Fidelity ledger* relatado para tudo o que foi cortado?
- [ ] O `<svg>` do diagrama tem `role="img"`, `aria-labelledby` que resolve, um `<title>` não vazio como primeiro filho, um `<desc>` não vazio e IDs prefixados por diagrama/variante?
- [ ] Formatos não HTML solicitados foram produzidos via [`export.md`](export.md), e não escritos à mão?
