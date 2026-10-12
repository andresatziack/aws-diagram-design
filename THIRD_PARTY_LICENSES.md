> Os textos das licenças são mantidos no idioma original, pois são juridicamente vinculantes.

# Licenças de terceiros

O aws-diagram-design é licenciado sob MIT (veja [`LICENSE`](LICENSE)). Ele é um derivado de
[cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) (MIT, © Cathryn Lavery),
com novo visual (re-skin) na marca AWS e com os AWS Architecture Icons oficiais incluídos. Ele redistribui
conteúdo das seguintes fontes de terceiros sob suas respectivas licenças.

## diagram-design (base upstream)

- **Licença:** MIT
- **Upstream:** https://github.com/cathrynlavery/diagram-design (versão base 2.3.2)
- **Usado em:** toda a estrutura da skill — `skills/aws-diagram-design/` (SKILL.md, referências, templates, exemplos, scripts), `commands/` e `scripts/`.

## AWS Architecture Icons

- **Licença / termos:** AWS Architecture Icons terms — https://aws.amazon.com/architecture/icons/
- **Copyright:** © Amazon Web Services, Inc. or its affiliates.
- **Release:** pacote de ícones 07312026.
- **Usado em:** `skills/aws-diagram-design/assets/aws-icons/` (812 SVGs: 305 de serviço, 466 de recurso, 15 de grupo, 26 de categoria), a galeria `assets/aws-icons.html` e `references/primitive-aws-icons.md`.

Fornecidos pela AWS para uso em diagramas de arquitetura conforme os termos acima. Os ícones não podem ser
alterados, recoloridos ou usados para sugerir patrocínio da AWS a produtos que não sejam da AWS.

## Amazon Ember (fonte — incluída)

- **Licença / termos:** Amazon Ember Licensing Guidelines — distribuídas junto com as fontes em
  `skills/aws-diagram-design/assets/fonts/Amazon-Ember-Licensing-Guidelines.pdf`.
- **Copyright:** © Amazon Technologies, Inc.
- **Origem:** download público de tipografia da Amazon,
  https://developer.amazon.com/en-US/alexa/branding/echo-guidelines/identity-guidelines/typography
  (`Amazon_Typefaces_Complete_Font_Set_Mar2020.zip`), redistribuído sem modificações.
- **Usado em:** `skills/aws-diagram-design/assets/fonts/` — webfonts woff2 (Amazon Ember
  300/400/400i/600/700/800, Amazon Ember Mono 400/700) referenciadas pelos diagramas gerados via
  `@font-face`, e TTFs para desktop instaláveis com `./install.sh fonts`.

Os diagramas sempre declaram a pilha de fallback completa
(`'Amazon Ember', 'Helvetica Neue', Helvetica, Arial, sans-serif`); ambientes que não conseguem
carregar os arquivos incluídos renderizam os fallbacks. Revise o PDF de diretrizes de licenciamento antes de
reutilizar as fontes fora desta skill.

## Geist Mono

- **Licença:** SIL Open Font License 1.1
- **Upstream:** https://vercel.com/font (carregada no momento da renderização a partir do Google Fonts)
- **Usado em:** apenas assets de exemplo legados pré-gerados (`skills/aws-diagram-design/assets/example-*.html`, construídos com skins anteriores) e a variante terminal opcional (opt-in) — carregada via `fonts.googleapis.com` no momento da renderização, não incluída. Diagramas gerados recentemente usam a **Amazon Ember Mono** incluída para todos os papéis mono.

## Tabler Icons

- **Licença:** MIT
- **Upstream:** https://github.com/tabler/tabler-icons
- **Usado em:** ícones de traço (stroked) incorporados em `skills/aws-diagram-design/references/primitive-icons.md` e `skills/aws-diagram-design/assets/icons.html` (categorias: Compute, People, Network, Data, Kubernetes, Action, DevOps, além dos contornos de marca em traço para Docker, Terraform, AWS, Azure, GitHub).

## Simple Icons

- **Licença:** CC0 1.0 Universal (Public Domain Dedication)
- **Upstream:** https://github.com/simple-icons/simple-icons
- **Usado em:** silhuetas de marca preenchidas incorporadas em `skills/aws-diagram-design/references/primitive-icons.md` e `skills/aws-diagram-design/assets/icons.html` (Kubernetes, Google Cloud, PostgreSQL, Nginx, Gitea, Keycloak, MinIO, Apache NiFi, Apache Airflow, Trino, Apache Superset, Jupyter, Python, R).

## log-z/logos

- **Licença:** MIT
- **Upstream:** https://github.com/log-z/logos/tree/main/website-logos
- **Usado em:** silhuetas de marca preenchidas incorporadas no mesmo conjunto de ícones genéricos — MySQL, Redis, StarRocks.

## Devicon

- **Licença:** MIT
- **Upstream:** https://github.com/devicons/devicon
- **Usado em:** os ícones de RStudio e SPSS incorporados no conjunto de ícones genéricos.

## Ícones de origem avulsa

O conjunto de ícones genéricos incorpora ícones de origem avulsa para SAS, do [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:SAS_logo_horiz.svg) (domínio público), e Stata, da [coleção IcePanel Technology Icons](https://icon.icepanel.io/Technology/svg/Stata.svg) publicada via techicons.dev.

## Marcas registradas

Os logotipos de marcas permanecem marcas registradas de seus respectivos proprietários. Sua inclusão neste conjunto de ícones
é apenas para uso em documentação e ilustração. A presença de uma marca neste repositório
não implica endosso, patrocínio ou afiliação.
