# Animação opcional

A animação explica um diagrama estático completo; ela nunca fornece significado ausente. Carregue esta referência apenas quando o movimento for explicitamente solicitado ou esclarecer de forma relevante ordem, acúmulo, avaliação, contenção ou propagação. Caso contrário, use o modo `none` e entregue HTML estático.

## Modos

Escolha um modo por figura com `data-motion-mode="none|reveal|step|loop"`.

| Modo | Comportamento | Controles / implementação | Uso |
|---|---|---|---|
| `none` | Figura completa e estável | Sem JavaScript | Padrão, impressão, captura de tela, exportação, fallback de movimento reduzido com controles de reprodução indisponíveis |
| `reveal` | Uma execução automática determinística que termina completa | Apenas CSS para ≤5s; caso contrário, use o controlador com escopo | Explicação ordenada curta; nunca reproduzir de novo automaticamente |
| `step` | Estados semânticos pausados | JS inline mínimo para Play, Pause, Replay, Previous, Next | Ensino, comparação, rastros de políticas |
| `loop` | Um token decorativo se repete sem mudar o significado | Padrão apenas CSS | Indicação discreta de fluxo; ciclo ≥3s |

Apenas `loop` se repete. Estado de fila, digitação, valores de campos, resultados de políticas, contenção e entradas de auditoria usam `reveal` ou `step` e terminam completos.

`reveal` é o único modo de reprodução automática sancionado: ele pode rodar uma vez no carregamento inicial quando o movimento foi explicitamente solicitado e depois permanece completo. Ele nunca reinicia ao reentrar na viewport nem sem uma ação explícita de Replay.

## Contrato de aprimoramento com prioridade ao estático

1. **A origem é completa.** Todo nó semântico, rótulo, conector, status e resultado fica visível no HTML/SVG antes do aprimoramento. Apenas seletores abaixo de `.motion-ready` podem ocultá-los ou transformá-los.
2. **Captura estável.** `data-frame="static"` inicial, `?motion=static`, impressão, sem JS e exportação de SVG autônomo expõem o quadro completo e ocultam controles/tokens decorativos. Não capture após um atraso arbitrário.
3. **O CSS é dono da apresentação.** Use transitions/keyframes de CSS para aparição e deslocamento. JavaScript inline mínimo é permitido apenas para vincular controles explícitos, atualizar atributos de etapa/estado, agendar etapas determinísticas e atualizar a região de status ao vivo dedicada. Nada de fetches, injeção de markup, medição de paths ou mutação de rótulos ou valores semânticos do diagrama.
4. **Um relógio.** Use `--motion-fast: 160ms`, `--motion-step: 480ms`, `--motion-hold: 720ms` e `--motion-total` ≤ `8000ms`; derive os atrasos de etapas inteiras. Nada de aleatoriedade, springs ou temporização por eventos de transição.
5. **Ordem explícita.** Marque os itens com `data-motion-item data-step="N"` para etapas inteiras de 1–8. A ordem do DOM segue a ordem narrativa. No máximo dois itens entram por etapa.
6. **Fim estável.** A conclusão expõe todos os itens e define `data-frame="end"`. Replay primeiro redefine para a etapa 0. Pause limpa o timer pendente e a retomada continua da mesma etapa.
7. **Estado com escopo.** Os controles operam no `[data-motion-root]` mais próximo; IDs, timers, regiões ao vivo e estado de etapa nunca cruzam os limites da figura.
8. **Inicialização à prova de falhas.** O JavaScript adiciona `.motion-ready` somente depois que os controles estão vinculados e a renderização inicial é bem-sucedida. Um erro de script antes desse ponto deixa a origem completa visível.

## Primitivos semânticos

Todo primitivo tem texto, contagem, símbolo, padrão ou contorno além da cor.

| Primitivo | Mecanismo | Resultado estático / com movimento reduzido | Limite |
|---|---|---|---|
| **Desenho de path** (path draw) | Path duplicado decorativo com `pathLength="1"` e dash offset animado | O conector base rotulado permanece visível | ≤2 paths; um ativo |
| **Revelação escalonada** (stage reveal) | `data-motion-item` + opacity/translate ≤8px | Todos os estágios visíveis | ≤8 etapas, 12 itens |
| **Contador de fila** (queue accumulation) | Slots estáveis; revelação de itens mais contagem numérica visível | Fila final e contagem visíveis | ≤5 itens; sem reordenar |
| **Digitação / preenchimento de campos** | String acessível completa; sobreposição decorativa recortada ou revelação de linhas rotuladas | Texto/campos completos visíveis uma vez | ≤32 caracteres digitados ou 6 campos |
| **Avaliação de política** (rule evaluation) | Linhas de regras ordenadas com status em texto e contorno na linha atual | Todo estado e resultado visíveis | 3–6 regras; 2 rastros |
| **Token de fluxo** | Token `aria-hidden` em um path fixo | Token oculto; o conector permanece | Um token; loop ≥3s |
| **Contenção** | Revela os filhos, depois um limite rotulado persistente | Filhos e limite visíveis | Uma transição de limite |
| **Acréscimo de auditoria** | Linhas cronológicas reveladas; timestamp/sequência estáveis | Log ordenado completo visível | ≤5 linhas acrescentadas |

Não anime coordenadas de layout, rotas de conectores, `viewBox`, dimensões de nós nem texto semântico. Evite zoom, parallax, bounce, shake, glow, partículas e piscar indefinido.

```css
:root {
  --motion-fast: 160ms;
  --motion-step: 480ms;
  --motion-hold: 720ms;
  --motion-total: 3600ms; /* five steps × hold; set this per diagram */
  --motion-ease: cubic-bezier(.2,.8,.2,1);
}
.motion-ready [data-motion-item] {
  opacity: .12;
  transform: translateY(8px);
  transition: opacity var(--motion-step) var(--motion-ease),
              transform var(--motion-step) var(--motion-ease);
}
.motion-ready [data-motion-item].is-visible,
.motion-ready[data-frame="end"] [data-motion-item] {
  opacity: 1;
  transform: none;
}
[data-motion-controls][hidden] { display: none !important; }
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
    scroll-behavior: auto !important;
  }
  [data-motion-item] { opacity: 1 !important; transform: none !important; }
  [data-motion-decorative] { display: none !important; }
  [data-motion-controls] { display: none !important; }
}
@media print {
  [data-motion-controls], [data-motion-decorative] { display: none !important; }
  [data-motion-item] { opacity: 1 !important; transform: none !important; }
}
```

## Controles interativos e teclado

Toda figura `step` interativa fornece botões nativos para **Play, Pause, Replay, Previous e Next**, fora do SVG. Use `data-motion-action="play|pause|replay|prev|next"`, alvos de ≥44×44px, foco visível, estado desabilitado para ações indisponíveis e `aria-pressed` para o estado de play/pause.

Quando o foco estiver dentro da raiz de movimento: `ArrowRight` avança, `ArrowLeft` volta, `Home` redefine, `End` completa, `Space` alterna play/pause quando o foco ainda não está em um controle nativo, e `R` sem modificadores reproduz de novo. Nunca intercepte `R` quando Control, Command ou Alt estiverem pressionados. Não capture teclas de inputs, links ou regiões não relacionadas. Nunca mova o foco quando o quadro mudar.

Forneça instruções visíveis e um `role="status" aria-live="polite" aria-atomic="true"` com escopo. Mantenha essa região ao vivo dentro da raiz de movimento, mas fora de `[data-motion-controls]`, para que ocultar os controles nos estados reduzido/estático não possa ocultar os anúncios. Anuncie ações do usuário como “Etapa 3 de 5: primeira divergência”; não anuncie cada quadro da reprodução automática. Os controles operam apenas no `[data-motion-root]` mais próximo.

Use [`assets/template-motion.html`](../assets/template-motion.html) em vez de inventar outro controlador. Seu controlador inline é o contrato de implementação executável: copie o corpo desse script literalmente. O linter de skin rejeita controladores modificados ou adicionais, mesmo quando carregam `data-diagram-controls`. Substitua o conteúdo do diagrama e os IDs prefixados pelo slug, mas preserve o controlador e seus atributos de estado/controle.

## Movimento reduzido, cor e acessibilidade

- `prefers-reduced-motion: reduce` inicializa no quadro estático completo, desabilita e oculta todos os controles de reprodução, oculta o movimento decorativo e expõe `data-motion-state="reduced"` mais um texto de status informando que a reprodução está indisponível. Nunca apresenta anúncios de etapas parciais ao lado de um quadro completo.
- O `<title>` e o `<desc>` do SVG descrevem o significado completo, não a animação. As instruções de interação permanecem como texto HTML visível.
- Sobreposições decorativas levam `aria-hidden="true" focusable="false"`. O texto semântico existe uma única vez na árvore de acessibilidade.
- O estado nunca é transmitido só por cor: políticas usam símbolo + `PASS/FAIL/SKIPPED/NOT REACHED`; filas mostram contagens; estágios ativos usam número/rótulo/contorno.
- Nada pisca nem muda de luminância mais de três vezes por segundo.

## Complexidade e temporização determinística

O movimento não aumenta o orçamento (limite) do diagrama estático: ≤8 etapas semânticas (alvo 3–6), ≤12 itens marcados, ≤2 revelações simultâneas, ≤2 paths desenhados, um loop de token de fluxo, transições de 160–600ms, pausas (holds) de 400–1200ms, translação ≤24px e 3–8s de reprodução automática no total.

Declare `data-step-count`; não infira etapas a partir de eventos de transição. Defina `--motion-total` como quantidade de etapas × `--motion-hold` e mantenha-o dentro do orçamento de 8 segundos. Use uma cadeia de `setTimeout` por raiz, derive sua pausa de `--motion-hold`, limpe-a em Pause/Replay/ocultação da página e imediatamente após renderizar a etapa final, e nunca use `setInterval` para reprodução semântica. Pause quando `document.visibilityState` se tornar hidden e não recupere o atraso depois. `?motion=step&step=N` pode expor um quadro exato de duração zero para regressão visual somente quando `N` for um inteiro não negativo em base 10 de 0 até `data-step-count`; valores ausentes, fracionários, negativos e acima do orçamento mantêm a reprodução normal.

O contrato de captura do estado final é síncrono: `?motion=static`, `<html data-motion="static">` ou o modo `none` expõe todos os itens semânticos, oculta controles e sobreposições decorativas e define `data-frame="static"`. Aguarde `document.fonts.ready` antes da captura. Duas capturas da mesma URL, viewport, fontes e escala de dispositivo devem ser idênticas pixel a pixel; atrasos aleatórios, IDs gerados, relógios e medição de paths em tempo de execução são proibidos.

## Exportação e verificação

As exportações PNG e SVG são artefatos estáticos do estado final, a menos que o usuário solicite explicitamente uma etapa nomeada. Antes da captura, abra `?motion=static`, aguarde `document.fonts.ready` e confirme `data-frame="static"`. A extração de SVG omite os controles e scripts HTML; o markup semântico visível na origem mantém o resultado completo.

Execute:

```bash
python3 scripts/verify-motion.py path/to/animated-diagram.html
```

O verificador checa as declarações de modo/estado, etapas contíguas, orçamentos de movimento, nomeação completa do SVG, visibilidade da origem sem JS, acessibilidade decorativa, o conjunto completo de controles, status ao vivo, CSS de movimento reduzido/impressão, tratamento de teclado, pausa ao ocultar a página, overrides estáticos/de teste delimitados, parada imediata na etapa final e identidade exata com o controlador canônico. Seus testes adversariais modificam o template canônico para provar que cada falha é rejeitada.

Depois, verifique em um navegador:

1. Desative o JavaScript: o diagrama completo permanece visível e com significado.
2. Emule `prefers-reduced-motion: reduce`: o estado final está completo, os controles de reprodução estão ocultos e desabilitados, e o status no DOM diz que a reprodução está indisponível.
3. Use apenas o teclado: Tab alcança cada controle nativo; Enter/Space os acionam; Left/Right/Home/End avançam etapas sem mover o foco.
4. Pause, retome e reproduza de novo duas vezes: a ordem e o estado final são idênticos.
5. Capture `?motion=static` duas vezes após `document.fonts.ready`: os pixels são estáveis.
6. Pré-visualização de impressão mais exportação PNG/SVG: controles e tokens decorativos estão ausentes, enquanto todos os rótulos e relações semânticas permanecem.

## Antipadrões

- Reprodução automática não solicitada, reprodução automática fora da única execução `reveal` sancionada, reentrada na viewport ou um loop semântico infinito.
- Um quadro em branco/parcial sem JS ou com movimento reduzido.
- Movimento que tenta salvar um diagrama estático denso demais ou sem rótulos.
- Pass/fail, fila cheia ou resultado codificados apenas por matiz.
- Scripts remotos, lógica geral de aplicação, geometria em tempo de execução ou texto semântico duplicado.
- Capturar após um atraso de relógio em vez de usar o override estático explícito.
