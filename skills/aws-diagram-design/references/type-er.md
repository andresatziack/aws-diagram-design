# ER / Modelo de dados

**Ideal para:** schemas de banco de dados, relacionamentos entre recursos de API, modelos de domínio.

## Convenções de layout
- Cada entidade é uma caixa com duas seções:
  - **Cabeçalho**: tag de tipo (`ENTITY`) + nome da entidade em Amazon Ember.
  - **Corpo**: lista de campos em Amazon Ember Mono, um por linha. PK prefixada com `#`, FK prefixada com `→`.
- Relacionamentos: linhas entre entidades com a cardinalidade em cada extremidade:
  - `1`, `N`, `0..1`, `1..*` em Amazon Ember Mono, 8px, posicionados a 10–12px da borda da entidade.
  - Rótulo opcional do relacionamento ("has", "belongs to") centralizado na linha.
- Mantenha entidades relacionadas próximas; organize o layout para que a maioria dos relacionamentos sejam linhas retas, não emaranhados.
- Coral na raiz do agregado ou na entidade central do modelo.

## Antipadrões
- Desenhar uma seta para cada FK em um modelo com dezenas delas — organize por cluster.
- Notação de cardinalidade inconsistente entre as extremidades do mesmo relacionamento.
- Campos preenchidos para forçar caixas de altura igual — a altura natural pelo conteúdo é adequada.

## Exemplos
- `assets/example-er.html` — minimal claro
- `assets/example-er-dark.html` — minimal escuro
- `assets/example-er-full.html` — editorial completo
