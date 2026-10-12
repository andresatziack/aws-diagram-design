# Guia de uso do Kiro (IDE · CLI)

Tanto a IDE quanto a CLI do Kiro leem nativamente Agent Skills no formato da Anthropic (SKILL.md). Basta colocar `skills/aws-diagram-design/` deste repositório em um diretório de skills, e o `plugin.json` da raiz (Agent Plugins 1.0.0) também permite instalá-lo como Power.

> **Instalação sem o `install.sh`:** veja [instalacao-kiro.md](instalacao-kiro.md), o guia passo a passo de instalação manual, com as adaptações para o Kiro (comandos de importação/exportação, hook de validação, Kiro Web). A revisão de segurança está em [revisao-seguranca.md](revisao-seguranca.md).

## Instalação

### Método 1 — Skill no nível do usuário (recomendado)

Funciona em todos os workspaces:

```bash
./install.sh kiro            # ~/.kiro/skills/aws-diagram-design/ 에 복사
./install.sh kiro --link     # 복사 대신 심링크 (저장소 수정이 즉시 반영, 개발용)
```

O primeiro comando copia a skill para `~/.kiro/skills/aws-diagram-design/`. O segundo cria um symlink em vez de copiar: as alterações no repositório valem na hora, o que é útil durante o desenvolvimento.

### Método 2 — Skill no workspace

Para usar em um projeto específico e compartilhar com o time por commit:

```bash
cd <프로젝트 루트>
/Users/sangbeom/Projects/aws-diagram-design/install.sh kiro --workspace   # .kiro/skills/ 에 복사
```

Execute a partir da raiz do seu projeto, trocando o caminho pelo local onde você clonou o repositório. A skill é copiada para `.kiro/skills/`. Se existir uma skill com o mesmo nome nos dois níveis, a do workspace tem prioridade.

### Método 3 — Interface de importação da IDE

Na IDE do Kiro, abra o painel **Agent Steering & Skills** → `+` → **Import a skill** → **Local folder** e selecione a pasta `skills/aws-diagram-design`. Para importar por URL do GitHub, aponte para a **subpasta da skill** (`…/tree/main/skills/aws-diagram-design`), não para a raiz do repositório. A importação copia os arquivos e não acompanha alterações futuras do repositório.

### Método 4 — Instalar como Power

Use Power se quiser instalação com um clique e atualização pelo painel. O manifesto é o `plugin.json` da raiz.

Painel **Powers** → **Add Custom Power** → **Import power from a folder** → selecione a raiz do clone (por exemplo `/Users/sangbeom/Projects/aws-diagram-design`). Na CLI V3, use `kiro-cli powers install <caminho>`. A documentação do Kiro não informa onde o Power instalado fica no disco. Depois de alterar o repositório, atualize com **Check for updates** → **Install updates** no Power.

Para uso comum, o Método 1 basta. Segundo a documentação do Kiro, o Power compensa sobretudo quando você distribui servidores MCP junto, e esta skill não tem nenhum.

## Uso

- O agente padrão (chat da IDE, `kiro-cli chat`) descobre a skill automaticamente. Pedidos como "desenhe um diagrama de arquitetura AWS" a ativam pela correspondência com a `description`.
- Chamada explícita: `/aws-diagram-design` (as skills aparecem como slash command na IDE, na CLI e no Kiro Web).
- O Kiro não lê os slash commands do Claude Code (`commands/`). Peça a importação de draw.io/Mermaid e a exportação em linguagem natural, e o roteamento do SKILL.md cuida do resto. Exemplos: "redesenhe este diagram.drawio para slide", "exporte o HTML que você acabou de gerar para PNG". Para ter comandos equivalentes no Kiro, use os steerings de `kiro/steering/` (veja [instalacao-kiro.md § 6.1](instalacao-kiro.md)).

### Atenção com agentes customizados

Agentes customizados (`kiro-cli chat --agent <nome>`) **não carregam skills por padrão**. Declare os recursos na configuração do agente (`.kiro/agents/<nome>.json`):

```json
{
  "resources": [
    "skill://~/.kiro/skills/**/SKILL.md",
    "skill://.kiro/skills/**/SKILL.md"
  ]
}
```

## Exportação e validação

O agente do Kiro executa comandos no terminal, então tudo funciona como no Claude Code:

```bash
pip install playwright && playwright install chromium   # PNG/SVG 내보내기 사전 준비 (1회)
python3 <스킬 경로>/scripts/self_check.py <생성된.html>  # 산출물 자체 점검
```

A primeira linha prepara a exportação PNG (uma única vez; prefira um ambiente virtual, veja [instalacao-kiro.md § 7](instalacao-kiro.md)). A segunda valida o diagrama gerado.

`drawio_extract.py` e `mermaid_extract.py` também são instalados na pasta da skill, e o agente os executa diretamente nas importações.

## Resumo das limitações

| Item | Detalhes |
|------|------|
| Slash commands | `commands/` do Claude Code não é suportado. Use o slash da skill (`/aws-diagram-design`), linguagem natural ou os steerings `kiro/steering/` |
| Importação pelo GitHub | Só aceita a URL da subpasta da skill; é uma cópia e não acompanha atualizações |
| Steering | Não use steering `always` com o conteúdo da skill, pois as referências entrariam em todo prompt. Os steerings deste repositório usam `inclusion: manual` e só são carregados quando chamados |
| Kiro Web | Só lê skills de `.kiro/skills/` do repositório. O upload em Settings tem limite de 25 arquivos por skill e não serve para esta skill |
| Fontes | Para renderizar com Amazon Ember fora do HTML, instale a fonte localmente (veja a seção de fontes do README) |
