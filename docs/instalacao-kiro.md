# Instalação manual no Kiro (sem `install.sh`)

Este guia instala a skill **aws-diagram-design** no Kiro copiando arquivos manualmente, para você saber exatamente o que entra na sua máquina. A revisão de segurança está em [revisao-seguranca.md](revisao-seguranca.md).

## 1. O que o Kiro usa deste repositório

| Caminho | Uso no Kiro |
|---|---|
| `skills/aws-diagram-design/` | **A skill.** É a única pasta obrigatória (SKILL.md + `references/` + `assets/` + `scripts/`). |
| `kiro/steering/*.md` | **Opcional, criado para o Kiro.** Substitui os slash commands do Claude Code (`/aws-import-drawio`, `/aws-import-mermaid`, `/aws-export-diagram`). |
| `kiro/hooks/aws-diagram-self-check.json` | **Opcional, criado para o Kiro.** Valida automaticamente diagramas salvos em `diagramas/`. |
| `plugin.json` | Manifesto, usado só se você instalar como **Power** (seção 4.4). |
| `commands/`, `.claude-plugin/` | Exclusivos do Claude Code. O Kiro ignora. |
| `install.sh`, `scripts/`, `docs/` | Não são necessários. |

Como o Kiro usa a skill:

1. Ao iniciar, carrega apenas o `name` e a `description` do `SKILL.md`.
2. Quando o seu pedido combina com a descrição (por exemplo, "desenhe um diagrama de arquitetura AWS"), carrega o SKILL.md completo.
3. Os arquivos de `references/` são lidos sob demanda, e os scripts de `scripts/` são executados pelo agente no terminal.

## 2. Pré-requisitos

- **Git**.
- **Python 3.9 ou superior** (`python3 --version`), para os scripts de validação e de importação. Usam só a biblioteca padrão; não é preciso `pip install`.
- *(Opcional)* **Playwright**, só para exportar PNG (seção 7).

## 3. Baixar e fixar a versão revisada

```bash
git clone https://github.com/andresatziack/aws-diagram-design.git
cd aws-diagram-design
git checkout 9e71e40ed1480dd0086a937ceebb32e9723ea58d   # commit revisado (original em inglês)
```

> Para usar a versão traduzida para português, faça checkout do commit desta tradução (ou da `main`, depois do merge) e anote o hash com `git rev-parse HEAD`. Ao atualizar no futuro, revise as mudanças antes de copiar: `git diff <hash-antigo>..<hash-novo> --stat`.

## 4. Escolha onde instalar

| Opção | Onde fica | Funciona em | Quando usar |
|---|---|---|---|
| **4.1 Workspace** (recomendado) | `<projeto>/.kiro/skills/aws-diagram-design/` | IDE, CLI e **Kiro Web** | Um projeto específico; compartilhar com o time via Git |
| **4.2 Usuário (global)** | `~/.kiro/skills/aws-diagram-design/` | IDE e CLI (não no Kiro Web) | Usar em todos os seus projetos locais |
| **4.3 Importar pela IDE** | cópia em uma das pastas acima | IDE | Prefere clicar a usar o terminal |
| **4.4 Power** | gerenciado pelo Kiro | IDE e CLI | Prefere instalar e atualizar pelo painel de Powers |

Se a skill existir no workspace e no usuário com o mesmo nome, **a do workspace tem prioridade**.

### 4.1 Workspace (por projeto)

Rode a partir da **raiz do seu projeto**. `CAMINHO_DO_CLONE` é a pasta clonada na seção 3.

**macOS / Linux:**

```bash
mkdir -p .kiro/skills
cp -R CAMINHO_DO_CLONE/skills/aws-diagram-design .kiro/skills/
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force .kiro\skills | Out-Null
Copy-Item -Recurse CAMINHO_DO_CLONE\skills\aws-diagram-design .kiro\skills\
```

Confira se o arquivo `.kiro/skills/aws-diagram-design/SKILL.md` existe. O nome da pasta precisa ser exatamente `aws-diagram-design`, igual ao `name:` do SKILL.md, senão o Kiro não reconhece a skill.

A skill tem cerca de 980 arquivos (7,7 MB), quase todos ícones SVG. Para compartilhar com o time, faça commit de `.kiro/skills/aws-diagram-design/`.

**Kiro Web:** o Kiro Web lê skills de `.kiro/skills/` **do repositório**. Faça o commit e o push dessa pasta, e a skill fica disponível nas sessões web do repositório. Não use o upload em *Settings > Skills* nem o *Configuration Sync* para esta skill: eles aceitam no máximo 25 arquivos por skill (50 por Power) e apenas arquivos de texto, e esta skill inclui fontes binárias.

### 4.2 Usuário (todos os projetos)

**macOS / Linux:**

```bash
mkdir -p ~/.kiro/skills
cp -R CAMINHO_DO_CLONE/skills/aws-diagram-design ~/.kiro/skills/
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force "$HOME\.kiro\skills" | Out-Null
Copy-Item -Recurse CAMINHO_DO_CLONE\skills\aws-diagram-design "$HOME\.kiro\skills\"
```

### 4.3 Importar pela IDE

1. No painel do Kiro, abra **Agent Steering & Skills**.
2. Clique em **+** → **Import a skill**.
3. Escolha a origem:
   - **Local folder:** selecione `CAMINHO_DO_CLONE/skills/aws-diagram-design`.
   - **GitHub:** a URL precisa apontar para a **subpasta** da skill, não para a raiz do repositório: `https://github.com/andresatziack/aws-diagram-design/tree/main/skills/aws-diagram-design`.

A importação **copia** os arquivos. Atualizações no repositório não chegam sozinhas; para atualizar, importe de novo.

### 4.4 Como Power

O `plugin.json` da raiz já é um manifesto válido de Power (formato Agent Plugins), e a skill está em `skills/aws-diagram-design/`.

- **IDE:** painel **Powers** → **Add Custom Power** → **Import power from a folder** → selecione `CAMINHO_DO_CLONE` (a raiz do clone, onde está o `plugin.json`) → **Install**. Para atualizar, use **Check for updates** → **Install updates**.
- **CLI (V3):** `kiro-cli powers install CAMINHO_DO_CLONE`, ou `/powers install CAMINHO_DO_CLONE` dentro do chat.

Esta skill não tem servidor MCP. Por isso, em uso normal, instalar como skill (4.1 ou 4.2) é mais simples e mostra com clareza o que foi copiado.

## 5. Verificar a instalação

1. Abra uma **nova** sessão de chat. As skills são descobertas ao iniciar a sessão.
2. **IDE:** digite `/` no chat. `aws-diagram-design` deve aparecer na lista, marcada como skill.
   **CLI:** rode `/context show` e confira se a skill aparece.
3. Peça, por exemplo: *"Desenhe um diagrama de arquitetura AWS: CloudFront → S3 e API Gateway → Lambda → DynamoDB. Salve em diagramas/serverless.html"*. Para chamar a skill explicitamente, use `/aws-diagram-design` seguido do pedido.
4. Teste os scripts locais (troque o caminho se tiver instalado no usuário):

```bash
python3 .kiro/skills/aws-diagram-design/scripts/self_check.py .kiro/skills/aws-diagram-design/assets/example-architecture.html
# Esperado: OK .kiro/skills/aws-diagram-design/assets/example-architecture.html
```

## 6. Adaptações para o Kiro (opcionais)

### 6.1 Comandos de importação e exportação

No Claude Code, este projeto oferece três slash commands (`commands/*.md`) que o Kiro **não lê**. Os pedidos em linguagem natural continuam funcionando ("redesenhe este arquivo.drawio para slide"), mas se quiser comandos com o comportamento fixo dos originais, copie os steerings que criei para o Kiro:

```bash
mkdir -p .kiro/steering
cp CAMINHO_DO_CLONE/kiro/steering/aws-*.md .kiro/steering/
# Para todos os projetos: copie para ~/.kiro/steering/
```

Eles usam `inclusion: manual`, ou seja, **só entram no contexto quando você chama**. Não ocupam contexto nos outros pedidos.

| Kiro | Equivale a (Claude Code) | Exemplo |
|---|---|---|
| `/aws-import-drawio` | `/aws-diagram-design:import-drawio` | `/aws-import-drawio legado.drawio, tamanho slide-16x9, detalhe simplified` |
| `/aws-import-mermaid` | `/aws-diagram-design:import-mermaid` | `/aws-import-mermaid docs/pipeline.mmd em html+png` |
| `/aws-export-diagram` | `/aws-diagram-design:export-diagram` | `/aws-export-diagram diagramas/serverless.html só svg` |

Você também pode chamá-los como `#aws-import-drawio` no meio de uma mensagem.

> Não transforme a skill inteira em steering `always`. Isso carregaria milhares de linhas de referência em todo pedido. A skill já é carregada sob demanda.

### 6.2 Validação automática ao salvar (hook)

Este hook roda o `self_check.py` sempre que o **agente** salva um `.html` dentro de `diagramas/`. Se houver problemas, a saída volta para o agente corrigir. Ele funciona na IDE e na CLI V3; hooks não rodam no Kiro Web.

```bash
mkdir -p .kiro/hooks
cp CAMINHO_DO_CLONE/kiro/hooks/aws-diagram-self-check.json .kiro/hooks/
```

Ajuste antes de usar:

- O hook espera a skill no workspace (`.kiro/skills/...`). Se instalou no usuário, troque o caminho no `command` por `~/.kiro/skills/aws-diagram-design/scripts/self_check.py`.
- Peça para o agente salvar os diagramas em `diagramas/`, ou mude o `matcher`.
- No Windows, troque `python3` por `python` ou `py`.

### 6.3 Agentes customizados (CLI)

O agente padrão encontra as skills sozinho. **Agentes customizados não carregam skills nem steering por padrão.** Declare os dois no `.kiro/agents/<nome>.json`:

```json
{
  "resources": [
    "skill://.kiro/skills/**/SKILL.md",
    "skill://~/.kiro/skills/**/SKILL.md",
    "file://.kiro/steering/**/*.md"
  ]
}
```

## 7. Exportar PNG (opcional)

A exportação SVG não precisa de nada extra. A exportação PNG usa o Chromium headless pelo Playwright, que **baixa programas da internet** (PyPI + ~150 MB do Chromium). A skill nunca instala isso automaticamente. Para isolar a instalação, use um ambiente virtual na raiz do projeto, com o nome que o steering `aws-export-diagram` procura:

```bash
python3 -m venv .venv-diagramas
.venv-diagramas/bin/pip install playwright          # Windows: .venv-diagramas\Scripts\pip install playwright
.venv-diagramas/bin/playwright install chromium     # Windows: .venv-diagramas\Scripts\playwright install chromium
echo ".venv-diagramas/" >> .gitignore
```

## 8. Fontes Amazon Ember (opcional)

Os diagramas funcionam sem instalar nada, com esta ordem de fontes: fonte local → woff2 incluído → Helvetica/Arial. Para ver a fonte Amazon Ember também fora do HTML (por exemplo em PNGs e editores), instale os TTF de `skills/aws-diagram-design/assets/fonts/ttf/` manualmente:

- **macOS:** `cp CAMINHO_DO_CLONE/skills/aws-diagram-design/assets/fonts/ttf/*.ttf ~/Library/Fonts/`
- **Linux:** `mkdir -p ~/.local/share/fonts/amazon-ember && cp CAMINHO_DO_CLONE/skills/aws-diagram-design/assets/fonts/ttf/*.ttf ~/.local/share/fonts/amazon-ember/ && fc-cache -f ~/.local/share/fonts/amazon-ember`
- **Windows:** selecione os `.ttf` no Explorador → botão direito → **Instalar** (ou **Instalar para todos os usuários**).

Leia `assets/fonts/Amazon-Ember-Licensing-Guidelines.pdf` antes de usar as fontes fora dos diagramas.

## 9. Atualizar e desinstalar

**Atualizar:**

```bash
cd CAMINHO_DO_CLONE
git fetch
git diff HEAD origin/main --stat   # revise antes
git checkout <novo-commit>
```

Em seguida, apague a pasta da skill instalada e copie de novo (seção 4). Se tiver alterado `references/style-guide.md` com o onboarding de marca, guarde uma cópia antes e reaplique depois.

**Desinstalar:** apague a pasta `.kiro/skills/aws-diagram-design/` (ou `~/.kiro/skills/aws-diagram-design/`), os arquivos `aws-*.md` de `.kiro/steering/` e `aws-diagram-self-check.json` de `.kiro/hooks/`. Se instalou como Power, desinstale pelo painel de Powers ou com `kiro-cli powers uninstall aws-diagram-design`.

## 10. Solução de problemas

| Sintoma | Causa provável e correção |
|---|---|
| A skill não aparece no `/` | A pasta não se chama `aws-diagram-design`, o `SKILL.md` não está no primeiro nível dela, ou a sessão é antiga. Abra uma nova sessão. |
| Funciona na IDE, mas não no Kiro Web | O Kiro Web não lê `~/.kiro/skills/`. Faça commit de `.kiro/skills/aws-diagram-design/` no repositório. |
| Agente customizado ignora a skill | Falta `resources` com `skill://` (seção 6.3). |
| `python3: command not found` | Instale Python 3.9+; no Windows use `py` ou `python`. |
| PNG não é gerado | Playwright ausente. Veja a seção 7; o agente vai mostrar a instrução e parar, de propósito. |
| O texto aparece em Helvetica | A fonte não carregou. Abra o HTML no navegador ou instale os TTF (seção 8). |
| Alterações na skill não surtem efeito | Existe uma cópia no workspace e outra no usuário, e você editou a errada. A do workspace tem prioridade. |
