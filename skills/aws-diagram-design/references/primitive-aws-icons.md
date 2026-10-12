# AWS Architecture Icons (primitivo)

Os AWS Architecture Icons oficiais, versão **07312026**, incluídos nesta skill em [`assets/aws-icons/`](../assets/aws-icons/). Use estes — e não a silhueta monocromática genérica `aws` — sempre que um diagrama citar serviços AWS específicos (EC2, Lambda, S3, Bedrock, …). Navegue visualmente em [`assets/aws-icons.html`](../assets/aws-icons.html); procure qualquer arquivo em [`assets/aws-icons/INDEX.md`](../assets/aws-icons/INDEX.md).

**812 ícones**: 305 de serviço, 466 de recurso, 15 de grupo, 26 de categoria.

## Quando usar cada conjunto

| Conjunto | Caminho | Tela (`viewBox`) | Anatomia | Usar para |
|---|---|---|---|---|
| **Service** | `service/<Category>/Arch_<Service>_48.svg` | `0 0 64 64` | Quadrado sólido na cor da categoria + glifo branco | Um serviço AWS nomeado como nó |
| **Resource** | `resource/<Category>/Res_<Service>_<Resource>_48.svg` | `0 0 48 48` | Glifo na cor da categoria, fundo transparente | Um recurso *dentro* de um serviço (instância EC2, função Lambda, objeto S3) e marcas genéricas (`General-Icons`: Users, Documents, Internet, …) |
| **Group** | `group/<Name>_32.svg` | `0 0 40 40` | Quadrado de cor sólida + glifo branco | Selo de canto na borda de uma zona/contêiner |
| **Category** | `category/Arch-Category_<Name>_48.svg` | `0 0 74 74` | Quadrado na cor da categoria + glifo branco | Uma categoria inteira, serviço ainda não escolhido |

Misturar é normal: ícones de serviço para nós, selos de grupo em contêineres, ícones de recurso para detalhes internos. Se o diagrama for sobre infraestrutura genérica (qualquer nuvem, nenhum serviço AWS nomeado), fique com o conjunto monocromático de [primitive-icons.md](primitive-icons.md).

## Cores de categoria (extraídas desta versão — a verdade de referência)

| Hex | Categorias |
|---|---|
| `#ED7100` laranja | Compute, Containers, Media Services, Blockchain, Quantum |
| `#8C4FFF` roxo | Analytics, Networking & Content Delivery, Games |
| `#C925D1` magenta | Databases, Developer Tools, Customer Enablement, Satellite |
| `#DD344C` vermelho | Security & Identity, Front-End Web & Mobile, Business Applications |
| `#7AA116` verde | Storage, IoT, Cloud Financial Management |
| `#01A88D` verde-azulado | Artificial Intelligence, Migration & Modernization, End User Computing |
| `#E7157B` rosa | Application Integration, Management & Governance |
| `#232F3E` squid ink | Ícones gerais, AWS Cloud |
| `#00A4A6` verde-azulado escuro | Region, sub-redes (apenas conjunto de grupos) |
| `#7D8998` cinza | Data center corporativo, servidor genérico (apenas conjunto de grupos) |

## Fluxo de busca

1. Verifique a tabela de serviços comuns abaixo — ela cobre a maioria dos diagramas.
2. Não está lá → procure o nome do serviço em `assets/aws-icons/INDEX.md`.
3. Ainda em dúvida → faça glob no diretório: `ls <skill-dir>/assets/aws-icons/service/*/`. A nomenclatura segue o pacote oficial: `Arch_Amazon-EC2_48.svg`, `Res_Amazon-EC2_Instance_48.svg`; ícones de recurso gerais terminam em `_48_Light.svg`.

Serviços renomeados que você precisa conhecer: SageMaker → `Arch_Amazon-SageMaker-AI`, QuickSight → `Arch_Amazon-Quick` (Quick Suite), ECS/EKS simples → `Arch_Amazon-Elastic-Container-Service` / `Arch_Amazon-Elastic-Kubernetes-Service`.

### Serviços comuns (relativos a `assets/aws-icons/`)

| Serviço | Caminho |
|---|---|
| EC2 | `service/Compute/Arch_Amazon-EC2_48.svg` |
| Lambda | `service/Compute/Arch_AWS-Lambda_48.svg` |
| Elastic Beanstalk | `service/Compute/Arch_AWS-Elastic-Beanstalk_48.svg` |
| App Runner | `service/Compute/Arch_AWS-App-Runner_48.svg` |
| Batch | `service/Compute/Arch_AWS-Batch_48.svg` |
| ECS | `service/Containers/Arch_Amazon-Elastic-Container-Service_48.svg` |
| EKS | `service/Containers/Arch_Amazon-Elastic-Kubernetes-Service_48.svg` |
| Fargate | `service/Containers/Arch_AWS-Fargate_48.svg` |
| ECR | `service/Containers/Arch_Amazon-Elastic-Container-Registry_48.svg` |
| S3 | `service/Storage/Arch_Amazon-Simple-Storage-Service_48.svg` |
| S3 Glacier | `service/Storage/Arch_Amazon-Simple-Storage-Service-Glacier_48.svg` |
| EFS | `service/Storage/Arch_Amazon-EFS_48.svg` |
| AWS Backup | `service/Storage/Arch_AWS-Backup_48.svg` |
| RDS | `service/Databases/Arch_Amazon-RDS_48.svg` |
| Aurora | `service/Databases/Arch_Amazon-Aurora_48.svg` |
| DynamoDB | `service/Databases/Arch_Amazon-DynamoDB_48.svg` |
| ElastiCache | `service/Databases/Arch_Amazon-ElastiCache_48.svg` |
| DMS | `service/Databases/Arch_AWS-Database-Migration-Service_48.svg` |
| API Gateway | `service/Networking-Content-Delivery/Arch_Amazon-API-Gateway_48.svg` |
| CloudFront | `service/Networking-Content-Delivery/Arch_Amazon-CloudFront_48.svg` |
| Route 53 | `service/Networking-Content-Delivery/Arch_Amazon-Route-53_48.svg` |
| ELB | `service/Networking-Content-Delivery/Arch_Elastic-Load-Balancing_48.svg` |
| VPC | `service/Networking-Content-Delivery/Arch_Amazon-Virtual-Private-Cloud_48.svg` |
| Transit Gateway | `service/Networking-Content-Delivery/Arch_AWS-Transit-Gateway_48.svg` |
| Direct Connect | `service/Networking-Content-Delivery/Arch_AWS-Direct-Connect_48.svg` |
| PrivateLink | `service/Networking-Content-Delivery/Arch_AWS-PrivateLink_48.svg` |
| SQS | `service/Application-Integration/Arch_Amazon-Simple-Queue-Service_48.svg` |
| SNS | `service/Application-Integration/Arch_Amazon-Simple-Notification-Service_48.svg` |
| EventBridge | `service/Application-Integration/Arch_Amazon-EventBridge_48.svg` |
| Step Functions | `service/Application-Integration/Arch_AWS-Step-Functions_48.svg` |
| Bedrock | `service/Artificial-Intelligence/Arch_Amazon-Bedrock_48.svg` |
| SageMaker AI | `service/Artificial-Intelligence/Arch_Amazon-SageMaker-AI_48.svg` |
| Amazon Q | `service/Artificial-Intelligence/Arch_Amazon-Q_48.svg` |
| Kinesis | `service/Analytics/Arch_Amazon-Kinesis_48.svg` |
| MSK | `service/Analytics/Arch_Amazon-Managed-Streaming-for-Apache-Kafka_48.svg` |
| Glue | `service/Analytics/Arch_AWS-Glue_48.svg` |
| Athena | `service/Analytics/Arch_Amazon-Athena_48.svg` |
| Redshift | `service/Analytics/Arch_Amazon-Redshift_48.svg` |
| EMR | `service/Analytics/Arch_Amazon-EMR_48.svg` |
| OpenSearch | `service/Analytics/Arch_Amazon-OpenSearch-Service_48.svg` |
| CloudWatch | `service/Management-Tools/Arch_Amazon-CloudWatch_48.svg` |
| CloudTrail | `service/Management-Tools/Arch_AWS-CloudTrail_48.svg` |
| Systems Manager | `service/Management-Tools/Arch_AWS-Systems-Manager_48.svg` |
| Organizations | `service/Management-Tools/Arch_AWS-Organizations_48.svg` |
| IAM | `service/Security-Identity/Arch_AWS-Identity-and-Access-Management_48.svg` |
| KMS | `service/Security-Identity/Arch_AWS-Key-Management-Service_48.svg` |
| Cognito | `service/Security-Identity/Arch_Amazon-Cognito_48.svg` |
| Secrets Manager | `service/Security-Identity/Arch_AWS-Secrets-Manager_48.svg` |
| WAF | `service/Security-Identity/Arch_AWS-WAF_48.svg` |
| Shield | `service/Security-Identity/Arch_AWS-Shield_48.svg` |
| GuardDuty | `service/Security-Identity/Arch_Amazon-GuardDuty_48.svg` |
| Users (genérico) | `resource/General-Icons/Res_Users_48_Light.svg` |
| User (genérico) | `resource/General-Icons/Res_User_48_Light.svg` |
| Internet (genérico) | `resource/General-Icons/Res_Internet_48_Light.svg` |
| Instância EC2 | `resource/Compute/Res_Amazon-EC2_Instance_48.svg` |
| Função Lambda | `resource/Compute/Res_AWS-Lambda_Lambda-Function_48.svg` |

## Embutindo um ícone

Leia o arquivo SVG, mantenha apenas o seu **`<g>` interno** (descarte a declaração XML, o invólucro `<svg>` externo e o `<title>`), remova os atributos `id` (dois ícones da mesma categoria colidem no id do grupo BG) e envolva-o em um grupo posicionado:

```svg
<!-- 24×24 service icon at (x,y): scale = 24 / 64 = 0.375 -->
<g transform="translate(X,Y) scale(0.375)" aria-hidden="true">
  <g stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
    <g fill="#ED7100"><rect x="0" y="0" width="64" height="64"></rect></g>
    <path d="…glyph…" fill="#FFFFFF"></path>
  </g>
</g>
```

Fatores de escala para renderização em 24px: serviço `24/64 = 0.375`, recurso `24/48 = 0.5`, selo de grupo `24/40 = 0.6`, categoria `24/74 ≈ 0.324` (use 32px → `32/74` se a proporção ímpar brigar com a grade de 4px). Arredonde o *posicionamento* para a grade de 4px; a geometria interna do ícone é isenta (assim como as larguras de traço).

### Padrão de nó com ícone de serviço

O ícone substitui a tag de tipo retangular do SKILL.md §6 — não empilhe os dois:

```svg
<rect x="X" y="Y" width="176" height="56" rx="6" fill="#ffffff"/>          <!-- opaque mask -->
<rect x="X" y="Y" width="176" height="56" rx="6" fill="#ffffff" stroke="#232F3E" stroke-width="1"/>
<g transform="translate(X+12, Y+16) scale(0.375)" aria-hidden="true">      <!-- 24px icon, left -->
  …inlined icon group…
</g>
<text x="X+48" y="Y+26" fill="#232F3E" font-size="12" font-weight="600"
      font-family="'Amazon Ember', 'Helvetica Neue', Arial, sans-serif">Order API</text>
<text x="X+48" y="Y+42" fill="#545B64" font-size="9"
      font-family="'Amazon Ember Mono', ui-monospace, monospace">Lambda · python3.13</text>
```

### Nós compactos (Compact nodes — tipos não arquiteturais)

**A regra do ícone se aplica a todos os tipos visuais, não apenas a Architecture.** Células de data-flow, etapas de processo, componentes de high-level, nós de DP-integration, atores de sequence, etapas de swimlane, camadas medallion — qualquer elemento cujo assunto seja um serviço AWS nomeado leva o ícone oficial.

Tipos cujos nós são pequenos demais para o padrão de ícone à esquerda (por exemplo, a célula de data-flow de 100×64, que já usa o canto esquerdo para um chip de papel) usam o **slot de canto de 16px**:

```svg
<!-- 16×16 service icon in the node's top-right corner: scale = 16 / 64 = 0.25 -->
<g transform="translate(NODE_X + NODE_W - 20, NODE_Y + 4) scale(0.25)" aria-hidden="true">
  …inlined icon group (ids stripped)…
</g>
```

Regras para o slot de canto:

- **16px é o piso** (regra 2 abaixo) — nunca renderize menor para espremer em uma célula apertada; em vez disso, remova o ícone dos nós secundários.
- Fatores de escala em 16px: serviço `16/64 = 0.25`, recurso `16/48 ≈ 0.333`, selo de grupo `16/40 = 0.4`.
- **Um ícone por nó** — o serviço principal do nó. Um nó que lista duas ferramentas (`Kinesis · DMS`) mostra a dominante; o sublabel já nomeia as demais.
- **Mantenha o título longe do ícone.** Com um título centralizado e um ícone de 16px no canto superior direito de um nó de 100px de largura, limite o título a ~13 caracteres; se o título não puder encolher, passe para o padrão de ícone à esquerda ou para um nó mais largo.
- Nós de assuntos não AWS (papéis, atores genéricos, ferramentas on-premises) não levam ícone AWS — misturar marcas monocromáticas genéricas ([primitive-icons.md](primitive-icons.md)) no mesmo diagrama não tem problema.

### Contêineres de grupo (zonas)

Os contêineres de zona AWS seguem a convenção oficial: **borda na cor do grupo, selo no canto superior esquerdo, rótulo à direita do selo**. O preenchimento é `none` ou uma tonalidade de ≤5% da cor da borda.

| Zona | Borda | Estilo | Arquivo do selo |
|---|---|---|---|
| AWS Cloud | `#242F3E` (claro) / `#232F3E` (escuro) | sólida | `group/AWS-Cloud_32.svg` (variante `_Dark` para skin escura) |
| Conta AWS | `#E7157B` | sólida | `group/AWS-Account_32.svg` |
| Region | `#00A4A6` | tracejada | `group/Region_32.svg` |
| Availability Zone | `#00A4A6` | tracejada | *nenhum — apenas rótulo de texto* |
| VPC | `#8C4FFF` | sólida | `group/Virtual-private-cloud-VPC_32.svg` |
| Sub-rede privada | `#00A4A6` | sólida | `group/Private-subnet_32.svg` |
| Sub-rede pública | `#7AA116` | sólida | `group/Public-subnet_32.svg` |
| Security group | `#DD344C` | tracejada | *nenhum — apenas rótulo de texto* |
| Auto Scaling group | `#ED7100` | tracejada | `group/Auto-Scaling-group_32.svg` (o selo no topo central também é oficial) |
| Data center corporativo | `#7D8998` | sólida | `group/Corporate-data-center_32.svg` |
| Conteúdo de instância EC2 | `#ED7100` | sólida | `group/EC2-instance-contents_32.svg` |
| Spot Fleet | `#ED7100` | sólida | `group/Spot-Fleet_32.svg` |
| Conteúdo de servidor | `#7D8998` | sólida | `group/Server-contents_32.svg` |

```svg
<!-- VPC container -->
<rect x="X" y="Y" width="W" height="H" fill="none" stroke="#8C4FFF" stroke-width="1"/>
<g transform="translate(X,Y) scale(0.6)" aria-hidden="true">…group/Virtual-private-cloud-VPC_32.svg inner g…</g>
<text x="X+32" y="Y+16" fill="#232F3E" font-size="12" font-weight="600"
      font-family="'Amazon Ember', 'Helvetica Neue', Arial, sans-serif">VPC · 10.0.0.0/16</text>
```

Os tracejados de zona usam `stroke-dasharray="4,4"`; zonas aninhadas ficam recuadas em ≥16px. Essas bordas de zona são *cores de identidade* e não contam no orçamento (limite) de accent — mas as regras de conectores do §6 e o orçamento de complexidade do §7 continuam valendo sem alteração.

## Regras (marca oficial + integração com a skill)

1. **Nunca recolora, reestilize, distorça, gire ou recorte um ícone oficial.** Se o diagrama precisar ser monocromático, use o conjunto genérico ([primitive-icons.md](primitive-icons.md)) em vez de tingir os ícones AWS.
2. **Renderização mínima de 16px**; 24–32px dentro dos nós é o ponto ideal. Mantenha a proporção 1:1.
3. **As cores de ícones e de bordas de zona são marcas de identidade, isentas do orçamento de accent.** O orçamento de 1–2 do `accent` (#FF9900) continua governando os elementos editoriais: traços de nós focais, setas destacadas, callouts.
4. **Não combine um ícone colorido com uma tag de tipo colorida** — o ícone já carrega o sinal da categoria. O texto do nó continua `ink`/`muted`.
5. **Nomes mais recentes**: esta versão renomeia vários serviços (veja Fluxo de busca). Rotule os nós com o nome atual do serviço, não com o nome histórico do ícone.
6. **Densidade**: ícones coloridos são visualmente chamativos. Acima de ~7 ícones, considere dividir o diagrama em zonas ou remover os ícones dos nós secundários (nós só com texto não têm problema).

## Atribuição de licença

AWS Architecture Icons © Amazon Web Services, Inc. — fornecidos pela AWS para uso em diagramas de arquitetura conforme os termos dos AWS Architecture Icons (https://aws.amazon.com/architecture/icons/). Não altere os ícones nem os use para sugerir patrocínio da AWS a produtos que não sejam da AWS.
