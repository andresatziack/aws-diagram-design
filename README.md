# aws-diagram-design

AWS 브랜드로 스킨된 에디토리얼 다이어그램 스킬. [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) 2.3.2를 기반으로 AWS 브랜드 커스터마이징(Amazon Ember 타이포그래피, squid ink `#232F3E` + smile orange `#EC7211`/`#FF9900` 팔레트, 공식 AWS Architecture Icons 812종 번들)을 적용해 독립 플러그인으로 재구성했습니다.

아키텍처, 플로차트, 시퀀스, 상태 머신, ER, 타임라인, 스윔레인, 쿼드런트, 레이더, 루프, 중첩, 트리, 조직도, 레이어 스택, 벤, 피라미드, 막대·선·간트·산점도, 하이레벨, 프로세스, 메달리온, 데이터 플로 등 27개 시각 타입을 자체 완결형 HTML/SVG/PNG로 생성하고, draw.io(.drawio)·Mermaid(.mmd) 원본을 지정한 크기·상세도로 다시 그립니다.

## 샘플

이 스킬로 생성한 아키텍처 다이어그램 예시입니다. 원본 HTML은 [docs/samples/](docs/samples/)에 있으며, 아래 SVG는 스킬의 export 절차로 추출한 것입니다.

### Serverless Web Application

CloudFront가 S3 정적 자산과 API Gateway → Lambda → DynamoDB 동적 경로를 분기하는 서버리스 웹 앱. ([HTML](docs/samples/serverless-web-app.html))

![Serverless Web Application 아키텍처 다이어그램](docs/samples/serverless-web-app.svg)

### VPC Three-Tier Web Service

퍼블릭 서브넷의 ALB가 프라이빗 서브넷의 ECS on Fargate로 전달하고, RDS·ElastiCache가 데이터 계층을 담당하는 VPC 3-티어 구성. 공식 AWS 그룹 컨테이너 규약(VPC·서브넷 배지)을 따릅니다. ([HTML](docs/samples/vpc-three-tier.html))

![VPC Three-Tier Web Service 아키텍처 다이어그램](docs/samples/vpc-three-tier.svg)

### Event-Driven Data Pipeline

Kinesis → Lambda → S3 데이터 레이크로 이어지는 스트리밍 수집과, Glue 카탈로그를 거쳐 Athena가 S3를 직접 쿼리하는 분석 경로. 교차 커넥터의 bridge/hop 프리미티브를 보여줍니다. ([HTML](docs/samples/event-driven-pipeline.html))

![Event-Driven Data Pipeline 아키텍처 다이어그램](docs/samples/event-driven-pipeline.svg)

### Multi-Region Network Backbone

멀티 계정·멀티 리전 네트워크 백본. us-east-1과 ap-northeast-2의 워크로드 VPC가 각 리전 Network account의 Transit Gateway에 attach되고, TGW가 AWS Cloud WAN 코어 네트워크로 피어링됩니다. 온프레미스 데이터센터는 Direct Connect(BGP)로 연결되며 Site-to-Site VPN이 백업 경로를 제공합니다. Region·Account·VPC·Corporate DC의 공식 그룹 컨테이너 규약을 모두 사용합니다. ([HTML](docs/samples/multi-region-network.html))

![Multi-Region Network Backbone 아키텍처 다이어그램](docs/samples/multi-region-network.svg)

### AWS Data Pipeline Flow

역할 스코프 데이터 플로(Data flow 타입). Data Admin이 Lake Formation으로 레이크를 거버넌스하고(트리거 점선), Data Engineer가 Kinesis·DMS로 수집해 S3 데이터 레이크에 적재 후 Glue로 ETL·카탈로그화하며, Analytics Engineer가 Redshift DW에 스타 스키마를 적재(포컬 핸드오프)하고 큐레이션한 뷰를 Data Consumer가 QuickSight로 소비합니다. 각 노드 하단 칩이 입·출력 페이로드 타입(스트림→데이터셋→테이블→리포트)의 변환 궤적을 보여줍니다. ([HTML](docs/samples/data-pipeline-flow.html))

![AWS Data Pipeline Flow 다이어그램](docs/samples/data-pipeline-flow.svg)

> Amazon Ember는 스킬에 번들되어 있습니다(`skills/aws-diagram-design/assets/fonts/`). 샘플 HTML은 `local('Amazon Ember')` → 번들 woff2 → Helvetica 순으로 로드하며, `./install.sh fonts`로 OS에 설치해 둘 수도 있습니다. 단, GitHub 이미지 렌더링은 외부 리소스를 차단하므로 아래 SVG의 폰트는 시스템 폰트로 대체 표시됩니다 — 정확한 타이포그래피는 HTML 원본을 브라우저에서 여세요.

## 지원 환경

| 환경 | 설치 방식 | 가이드 |
|------|-----------|--------|
| Claude Code | 플러그인(마켓플레이스) 또는 스킬 단독 | [docs/guide-claude-code.md](docs/guide-claude-code.md) |
| Kiro (IDE / CLI) | Agent Skill 폴더 또는 Power | [docs/guide-kiro.md](docs/guide-kiro.md) |

두 환경 모두 `skills/aws-diagram-design/` 하나를 공유합니다. Kiro는 Anthropic 형식 Agent Skills를 그대로 읽으므로 별도 변형이 필요 없습니다.

## 빠른 설치

```bash
./install.sh claude-code     # Claude Code 설치 절차 안내 출력
./install.sh kiro            # ~/.kiro/skills/ 에 복사 (--workspace: .kiro/skills/, --link: 심링크)
./install.sh fonts           # 번들된 Amazon Ember TTF를 OS 폰트 디렉터리에 설치
```

## 저장소 구조

```
aws-diagram-design/
├── plugin.json                     # Agent Plugins 1.0.0 매니페스트 (Kiro Power용, 루트 위치)
├── .claude-plugin/
│   ├── plugin.json                 # Claude Code 플러그인 매니페스트
│   └── marketplace.json            # Claude Code 마켓플레이스 카탈로그
├── commands/                       # Claude Code 슬래시 커맨드 3종
│   ├── export-diagram.md           #   /aws-diagram-design:export-diagram
│   ├── import-drawio.md            #   /aws-diagram-design:import-drawio
│   └── import-mermaid.md           #   /aws-diagram-design:import-mermaid
├── skills/aws-diagram-design/      # 정본 스킬 (세 환경 공용)
│   ├── SKILL.md                    #   설계 시스템 + 27개 타입 라우팅
│   ├── references/                 #   40개 레퍼런스 (style-guide, type-*, primitive-aws-icons 등)
│   ├── assets/                     #   템플릿·예제 HTML, aws-icons/ 812 SVG, 아이콘 갤러리, fonts/ Amazon Ember
│   └── scripts/                    #   self_check.py, verify-geometry.py, drawio/mermaid 추출기, 아이콘 빌더
├── scripts/                        # 저장소 수준 검증 스크립트 (verify-geometry, verify-motion) + fixtures/
├── docs/                           # 환경별 한글 가이드
└── install.sh                      # 타깃별 설치 스크립트
```

## 라이선스

MIT. 서드파티 고지는 [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) 참고 — AWS Architecture Icons는 [AWS 아이콘 이용 약관](https://aws.amazon.com/architecture/icons/)을 따르며 변형·재채색이 금지됩니다.
