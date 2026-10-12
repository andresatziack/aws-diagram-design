# Padrões semânticos

Padrões semânticos descrevem **o que um sistema faz**; os 27 tipos visuais descrevem **como a informação é organizada**. Escolha primeiro um padrão quando comportamento, estado, aplicação de controles ou risco forem essenciais, e então use o tipo visual mais próximo como gramática de layout. Se nenhum padrão corresponder, escolha um tipo visual diretamente.

Use um padrão principal por figura. Um segundo padrão pode fornecer no máximo uma primitiva de apoio; se ambos precisarem de tratamento completo, separe visão geral e detalhe. Rótulos e resultados devem permanecer completos em um quadro estático.

## Tabela de roteamento

| O leitor precisa entender… | Padrão semântico | Tipo visual mais próximo |
|---|---|---|
| Muitas chegadas competindo por capacidade de atendimento finita | **Fila de convergência (fan-in) / gargalo** | Data flow |
| Perguntas, entradas, controles e saídas repetidos ao longo das etapas | **Framework de etapas com slots semânticos** | Process |
| Uma conversa solta se tornando um registro estruturado durável | **Entrada não estruturada → artefato estruturado** | Data flow |
| Por que duas decisões de política diferem e onde elas divergem pela primeira vez | **Traços pareados de avaliação de política** | Flowchart |
| Quais rotas cruzam uma fronteira de confiança e quais rotas são bloqueadas | **Caminho pavimentado seguro (secure paved road)** | Architecture |
| Quais controles se aplicam em cada superfície de aplicação | **Catálogo de governança / controles** | Layer stack |
| Como as defesas reduzem o risco e que risco permanece | **Camadas de segurança compensatórias** | Layer stack |

## 1. Fila de convergência (fan-in) / gargalo

**Gatilhos de seleção:** Vários produtores convergem para um único revisor, serviço, portão ou recurso restrito; a história depende de taxa de chegada, profundidade da fila, espera, capacidade ou contrapressão (backpressure).

**Primitivas obrigatórias:** Fontes distintas; entrada distribuída em leque; uma fila ordenada com slots visíveis e contagem; um rótulo de capacidade/taxa de atendimento; um único ponto de atendimento restrito; resultados admitidos e adiados/rejeitados. Rotule com unidades (`8/hour`, `3 slots`), não apenas “alto”.

**Orçamento (limite) de complexidade:** ≤5 fontes, ≤5 slots de fila, um gargalo, dois resultados e ≤9 nós primários. Agregue o excesso de fontes como um grupo nomeado.

**Antipadrões:** Pipeline de largura uniforme que esconde a contenção; setas mescladas antes que possam ser rastreadas; capacidade sugerida apenas pelo tamanho da caixa; amontoado decorativo; animação que altera a ordem dos itens; vermelho sozinho significando sobrecarga.

**Fallback estático:** Mostre a fila final representativa, a contagem/capacidade numérica, o rótulo do gargalo e ambos os caminhos de resultado. Um quadro estático deve revelar por que o trabalho espera.

**Tipo visual mais próximo:** **Data flow** por padrão; use **Process** quando as etapas de atendimento, e não as fontes, predominarem.

## 2. Framework de etapas com slots semânticos

**Gatilhos de seleção:** Um ciclo de vida ou modelo operacional repete as mesmas perguntas semânticas ao longo das etapas, comumente Pergunta, Entrada, Governança e Saída. A comparabilidade entre etapas importa mais do que o tempo das mensagens.

**Primitivas obrigatórias:** Cabeçalhos de etapa ordenados; uma grade de slots consistente; slots vazios/não aplicáveis explícitos; passagem de etapa para etapa; rótulos de slot estáveis; uma saída principal por etapa. Preserve a ordem dos slots em todas as etapas.

**Orçamento (limite) de complexidade:** 3–6 etapas, 3–4 tipos de slot, ≤20 células preenchidas, ≤2 linhas por célula. Separe o detalhe quando uma célula precisar de prosa.

**Antipadrões:** Cada etapa inventa um layout interno diferente; significado do slot codificado pela posição sem rótulos; falsa precisão com dezenas de células; confundir a ordem das etapas com raias de responsabilidade; encolher o texto para manter uma única tela.

**Fallback estático:** Renderize a matriz completa etapa × slot com as passagens e entradas explícitas `—` ou `Not applicable`. Não dependa de revelação gradual para ensinar o esquema.

**Tipo visual mais próximo:** **Process**; use **Swimlane** somente quando as linhas repetidas representarem responsáveis, e não slots semânticos.

## 3. Entrada não estruturada → artefato estruturado

**Gatilhos de seleção:** Diálogo, anotações, prompts ou um pedido prolixo são elicitados, normalizados e escritos em um brief, ticket, registro, esquema ou outro artefato estruturado durável.

**Primitivas obrigatórias:** Enunciado(s) de origem; perguntas de esclarecimento; pares campo/valor extraídos; uma transformação nomeada; a fronteira do artefato durável; links de proveniência de declarações representativas para campos; estado ausente/desconhecido.

**Orçamento (limite) de complexidade:** ≤4 trocas, ≤6 campos de artefato, uma transformação e ≤3 links de proveniência. Mostre conteúdo representativo, não uma transcrição.

**Antipadrões:** Brilho de “mágica de IA” entre duas caixas; artefato mostrado como mais um balão de chat; campos aparecendo sem fontes; inventar certeza para fatos ausentes; animação de digitação como a única cópia legível.

**Fallback estático:** Mostre um trecho curto da fonte ao lado do artefato rotulado completo, com pelo menos um mapeamento de proveniência e quaisquer campos desconhecidos visíveis.

**Tipo visual mais próximo:** **Data flow**; use **Process** quando a elicitação tiver vários portões ordenados.

## 4. Traços pareados de avaliação de política

**Gatilhos de seleção:** Duas solicitações de resto semelhantes chegam a resultados diferentes; o leitor precisa do estado regra a regra `PASS`, `FAIL`, `SKIPPED` ou `NOT REACHED` e da primeira divergência.

**Primitivas obrigatórias:** As mesmas regras ordenadas em ambos os traços; texto de status explícito mais símbolo/forma; as entradas que diferem; resultados finais; um marcador rotulado da primeira divergência; uma distinção entre `SKIPPED` (fluxo aplicável intencionalmente ignorado) e `NOT REACHED` (a avaliação parou antes).

**Orçamento (limite) de complexidade:** Exatamente 2 traços, 3–6 regras, uma primeira divergência, ≤12 células de status e um resultado por traço. Mova a prosa das regras para notas se os rótulos excederem uma linha.

**Antipadrões:** Comparar dois fluxos ordenados de forma independente; pontos verdes/vermelhos sem palavras; tratar skipped e not-reached como sinônimos; destacar todas as diferenças; continuar um traço negado como se as regras seguintes tivessem sido executadas.

**Fallback estático:** Mostre todos os estados das regras e ambos os resultados de uma vez; use um colchete/linha persistente e um rótulo para a primeira divergência.

**Tipo visual mais próximo:** **Flowchart** para lógica de decisão ordenada; use **Sequence** somente quando mensagens entre atores e o tempo também forem essenciais.

## 5. Caminho pavimentado seguro (secure paved road)

**Gatilhos de seleção:** Uma arquitetura suportada cria uma rota delimitada desde a entrada/build até o deploy; fronteiras de confiança, momentos privilegiados, entrada permitida, entrada proibida e caminhos de deploy aprovados versus bloqueados são o ponto central.

**Primitivas obrigatórias:** Fronteiras de confiança rotuladas; atores e identidades; entrada permitida com um rótulo de texto positivo; entrada proibida terminando na fronteira; caminho de deploy aprovado; caminho de desvio bloqueado; portão privilegiado; runtime isolado; destino de auditoria. Use estilos de linha e símbolos de parada diferentes, além da cor.

**Orçamento (limite) de complexidade:** ≤3 zonas de confiança, ≤8 componentes, ≤10 caminhos, ≤2 caminhos proibidos e um portão privilegiado. Separe o detalhe dos controles em uma figura de catálogo.

**Antipadrões:** Caixa tracejada chamada “segurança” sem semântica de rota; seta proibida atravessando para dentro da zona protegida; segredos ou identidade sugeridos, mas não rotulados; todos os componentes estilizados como confiáveis; um caminho de desvio que visualmente volta a se juntar à rota aprovada.

**Fallback estático:** Renderize todas as fronteiras e ambas as rotas permitidas/proibidas. Caminhos bloqueados devem parar visivelmente antes da entrada ou do deploy.

**Tipo visual mais próximo:** **Architecture**.

## 6. Catálogo de governança / controles

**Gatilhos de seleção:** Um inventário de controles precisa ser entendido pelo local onde é aplicado: autoria, workspace, merge/CI, deploy/runtime ou outra superfície nomeada. Um único checklist esconderia esses pontos de aplicação.

**Primitivas obrigatórias:** Grupos por superfície de aplicação; controles nomeados; ator que aplica (`code`, `platform`, `human`); momento (`write`, `merge`, `deploy`, `run`); possibilidade de contorno ou rota de exceção; notação de cobertura/lacuna.

**Orçamento (limite) de complexidade:** 3–5 superfícies, 3–7 controles por superfície, ≤24 controles no total e ≤3 atributos por controle. Resuma contagens somente quando a lista de itens existir em outro lugar.

**Antipadrões:** 35 pílulas minúsculas; agrupar por temas vagos em vez de ponto de aplicação; misturar aspirações com controles efetivamente aplicados; ícones sem nomes de controle; alegar defesa em profundidade sem mostrar a cobertura das superfícies.

**Fallback estático:** Mostre o catálogo agrupado completo com cabeçalhos de superfície e rótulos de texto para ator e momento de aplicação; preserve lacunas e exceções.

**Tipo visual mais próximo:** **Layer stack**; use **DP security matrix** quando permissões de papéis, e não superfícies de aplicação, forem a comparação dominante.

## 7. Camadas de segurança compensatórias

**Gatilhos de seleção:** Nenhuma camada é perfeita; cada defesa cobre uma falha deixada pela camada anterior, e o risco residual deve visivelmente se estreitar, ser transferido ou permanecer ao longo da pilha.

**Primitivas obrigatórias:** Entrada ordenada de ameaça/risco; camadas defensivas nomeadas; a mitigação de cada camada; limitação ou escape explícito; portador do risco residual entre as camadas; risco residual final e consequência/resposta. Use rótulos ou medidas decrescentes, nunca apenas área.

**Orçamento (limite) de complexidade:** 3–5 camadas, um fio principal de risco, ≤2 mitigações por camada e uma declaração final de risco residual. Separe múltiplas ameaças não relacionadas em figuras distintas.

**Antipadrões:** Sugerir que a camada final zera o risco; placas opacas iguais sem propagação; tratar auditoria como prevenção; encolher formas sem significado numérico ou verbal; inverter a ordem prevenção/detecção/recuperação sem explicação.

**Fallback estático:** Mostre a cadeia de propagação completa: risco inicial → mitigação → risco escapado em cada camada → risco residual final e resposta.

**Tipo visual mais próximo:** **Layer stack**; use **Nested** quando fronteiras de contenção, e não a compensação ordenada, carregarem o significado.

## Regras de composição

- O padrão semântico pode especializar primitivas de status, fronteira, fila ou propagação; o tipo selecionado continua sendo dono do eixo da página, da gramática de conectores, do espaçamento e dos limites específicos do tipo.
- Aplique o mais restritivo entre o orçamento do padrão e o orçamento do tipo visual. Células/status semânticos não são permissão para exceder a meta de nove nós da visão geral.
- Use texto estável para estados e resultados. Cor, movimento e posição reforçam o significado, mas nunca o carregam sozinhos.
- Animação opcional é uma camada de apresentação, não outro padrão. Carregue [`animation.md`](animation.md) somente quando movimento for solicitado ou esclarecer materialmente uma mudança ordenada.
