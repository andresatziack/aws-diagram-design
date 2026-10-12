# Guia de uso do Claude Code

## Instalação

### Método 1 — Plugin (recomendado)

A skill e os 3 slash commands são instalados juntos. Em uma sessão do Claude Code:

```
/plugin marketplace add /Users/sangbeom/Projects/aws-diagram-design
/plugin install aws-diagram-design@aws-diagram-design
```

Após a instalação, siga as instruções para executar `/reload-plugins` ou reinicie a sessão. Depois de publicar no GitHub, você pode adicioná-lo no formato `owner/aws-diagram-design` em vez do caminho local.

### Método 2 — Apenas a skill

Para usar só a skill, sem os slash commands, basta um link simbólico:

```bash
ln -s /Users/sangbeom/Projects/aws-diagram-design/skills/aws-diagram-design ~/.claude/skills/aws-diagram-design
```

Como é um link simbólico, alterações no repositório são refletidas imediatamente. Durante o desenvolvimento, este método é mais prático.

## Limpeza do plugin diagram-design existente

Este plugin substitui a configuração "diagram-design da marketplace + overlay da skin AWS". Se os dois ficarem ativos ao mesmo tempo, duas skills com condições de ativação sobrepostas competem entre si, então faça a seguinte limpeza.

1. Remova o plugin existente: `/plugin` → diagram-design → uninstall. Se você apenas desativar (disable), a árvore de cache (`~/.claude/plugins/cache/diagram-design/`) permanece e o hook SessionStart continua tendo essa árvore como alvo — por enquanto ele fica silencioso só porque existe o marcador da skin (`.aws-skin-applied`), mas se você aumentar o `SKIN_VERSION` do overlay, ele volta a aplicar o patch no plugin desativado.
2. Limpe o hook SessionStart: o hook `apply_aws_skin.py` em `~/.claude/settings.json` não falha (sempre exit 0), mas após o uninstall do plugin ele imprime a linha `[aws-diagram-skin] diagram-design plugin not found; nothing to do.` a cada sessão. Remover a entrada do hook também elimina essa saída. É melhor manter o diretório `~/.claude/aws-diagram-skin/`, pois ele pode ser reutilizado para reaplicar a skin AWS em uma nova versão do upstream.

## Uso

### Ativação automática

Pedidos como "AWS 아키텍처 다이어그램 그려줘", "구성도 만들어줘" ou "이 흐름을 플로차트로" ativam a skill automaticamente. Os gatilhos em inglês e em coreano estão todos na description do SKILL.md.

```
빗썸 시세 수집 파이프라인 아키텍처 다이어그램 그려줘.
API Gateway → Lambda → Kinesis → S3, 모니터링은 CloudWatch.
```

Ao mencionar serviços AWS, os ícones oficiais são inseridos automaticamente, e os limites de VPC, sub-rede e conta são desenhados conforme a convenção oficial de grupos (cor da borda + selo no canto superior esquerdo).

### Slash commands

| Comando | Função |
|--------|------|
| `/aws-diagram-design:export-diagram <html> [--svg-only\|--png-only] [--scale=N] [--output=<path>]` | Exportar o HTML gerado para .svg/.png |
| `/aws-diagram-design:import-drawio <file.drawio>` | Redesenhar um original draw.io no estilo editorial |
| `/aws-diagram-design:import-mermaid <file.mmd>` | Redesenhar um original Mermaid |

Os dois comandos de importação aceitam também `--format=html|svg|png|html+png`, `--size=<프리셋>`, `--detail=faithful|balanced|simplified`, `--audience=engineer|mixed|executive`, `--variant=light|dark|full`, `--output=<경로>` e seleção de página (`--page`/`--diagram`). A referência para a assinatura completa é o argument-hint de cada `commands/*.md`.

### Preparação para exportar PNG/SVG

A exportação usa o Chromium headless. Basta instalar uma vez:

```bash
pip install playwright && playwright install chromium
```

## Verificação dos artefatos

Com o repositório em checkout, você pode verificar os diagramas gerados com scripts:

```bash
python3 skills/aws-diagram-design/scripts/self_check.py <생성된.html>   # 접근성·단일 파일 계약
python3 scripts/verify-geometry.py <생성된.html>                        # 라벨 마스크-노드 겹침
python3 scripts/verify-motion.py <생성된.html>                          # 모션 사용 시
```

Mesmo ao usar a skill instalada, o `self_check.py` é distribuído junto no diretório da skill, então pode ser executado da mesma forma.

## Fontes

A Amazon Ember precisa estar instalada localmente para a renderização com a fonte especificada. Se o resultado no navegador ou nas capturas de tela aparecer em Helvetica, a fonte não está instalada. Para verificar a instalação, consulte a seção de fontes do README.
