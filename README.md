# aws-diagram-design

AWS 브랜드로 스킨된 에디토리얼 다이어그램 스킬. [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) 2.3.2를 기반으로 AWS 브랜드 커스터마이징(Amazon Ember 타이포그래피, squid ink `#232F3E` + smile orange `#EC7211`/`#FF9900` 팔레트, 공식 AWS Architecture Icons 812종 번들)을 적용해 독립 플러그인으로 재구성했습니다.

아키텍처, 플로차트, 시퀀스, 상태 머신, ER, 타임라인, 스윔레인, 쿼드런트, 레이더, 루프, 중첩, 트리, 조직도, 레이어 스택, 벤, 피라미드, 막대·선·간트·산점도, 하이레벨, 프로세스, 메달리온, 데이터 플로 등 27개 시각 타입을 자체 완결형 HTML/SVG/PNG로 생성하고, draw.io(.drawio)·Mermaid(.mmd) 원본을 지정한 크기·상세도로 다시 그립니다.

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
│   ├── assets/                     #   템플릿·예제 HTML, aws-icons/ 812 SVG, 아이콘 갤러리
│   └── scripts/                    #   self_check.py, verify-geometry.py, drawio/mermaid 추출기, 아이콘 빌더
├── scripts/                        # 저장소 수준 검증 스크립트 (verify-geometry, verify-motion) + fixtures/
├── docs/                           # 환경별 한글 가이드
└── install.sh                      # 타깃별 설치 스크립트
```

## 라이선스

MIT. 서드파티 고지는 [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) 참고 — AWS Architecture Icons는 [AWS 아이콘 이용 약관](https://aws.amazon.com/architecture/icons/)을 따르며 변형·재채색이 금지됩니다.
