# Revisão de segurança — aws-diagram-design

| Item | Valor |
|---|---|
| Data da revisão | 12/10/2026 |
| Repositório | `andresatziack/aws-diagram-design` (fork de `masangbeom/aws-diagram-design`) |
| Commit revisado | `9e71e40ed1480dd0086a937ceebb32e9723ea58d` (branch `main`) |
| Situação do fork | Idêntico ao upstream nesse commit (0 commits à frente e 0 atrás) |
| SHA-256 do `install.sh` | `c34584b47a647cafd267ebb77f031f88cd263f5223afa0e4772bb6e47b86fcfa` |
| Arquivos versionados | 1.010 (817 SVG, 108 HTML, 52 Markdown, 8 Python, 16 fontes, 1 shell, 1 PDF, JSON) |

## Veredito

**Não encontrei código malicioso.** O `install.sh` **não baixa nada da internet**: ele apenas copia (ou cria um symlink para) a pasta `skills/aws-diagram-design/` e, opcionalmente, copia as fontes TTF incluídas no repositório para a pasta de fontes do seu usuário. Não usa `sudo`, não altera `PATH`, `.bashrc`/`.zshrc`, crontab, credenciais ou configurações do sistema.

O risco residual é baixo e vem de três pontos descritos abaixo: (1) os exemplos HTML carregam fontes do Google Fonts ao serem abertos no navegador; (2) a exportação PNG exige instalar o Playwright/Chromium (download externo, feito por você, nunca automaticamente); (3) como toda skill de agente, ela instrui o agente a executar scripts Python locais e, no onboarding por URL, a acessar sites externos.

## 1. `install.sh` — análise linha a linha

| Comando | O que faz | Escreve em | Rede |
|---|---|---|---|
| `./install.sh claude-code` | Só **imprime** instruções (`cat <<EOF`). Não executa nada. | — | Não |
| `./install.sh kiro` | `cp -R skills/aws-diagram-design ~/.kiro/skills/aws-diagram-design` | `~/.kiro/skills/aws-diagram-design` | Não |
| `./install.sh kiro --workspace` | Igual, mas no diretório atual (`$PWD`) | `./.kiro/skills/aws-diagram-design` | Não |
| `./install.sh kiro --link` | `ln -s` (symlink) em vez de cópia | idem | Não |
| `./install.sh fonts` | `cp ttf/*.ttf` + `fc-cache -f` (só no Linux, se existir) | macOS: `~/Library/Fonts`; Linux: `~/.local/share/fonts/amazon-ember` | Não |
| `--force` | `rm -rf` **somente** no destino fixo acima, antes de copiar de novo | idem | Não |

Pontos positivos: `set -euo pipefail`, opções desconhecidas abortam o script, caminhos derivados do próprio repositório (`BASH_SOURCE`), nenhuma expansão de entrada do usuário em comandos.

Cuidados (não são vulnerabilidades):

- `--force` apaga o destino antes de copiar — se você editou a skill instalada, perderá as edições.
- `--link` faz qualquer `git pull` futuro entrar em vigor **sem nova revisão**. Para manter controle, prefira cópia e reavalie a cada atualização.
- `--workspace` usa o diretório em que você está; rode a partir da raiz do projeto certo.

## 2. Scripts Python (6 na skill + 2 na raiz)

`self_check.py`, `verify-geometry.py`, `drawio_extract.py`, `mermaid_extract.py`, `build_aws_icons.py`, `scripts/verify-motion.py`, `scripts/verify-geometry.py`.

- **Só biblioteca padrão** (`argparse`, `re`, `json`, `html.parser`, `xml.etree`, `zlib`, `base64`, `struct`, `pathlib`, `urllib.parse`). Nenhuma dependência de terceiros.
- **Nenhum** uso de rede (`socket`, `urllib.request`, `http.client`, `requests`), `subprocess`, `os.system`, `eval`, `exec`, `pickle`, `marshal`, `ctypes` ou `__import__`.
- `urllib.parse` é usado apenas para **analisar** URLs (o `self_check.py` rejeita referências remotas nos diagramas, exceto `https://fonts.googleapis.com/css2`).
- `base64`/`zlib` em `drawio_extract.py` servem para decodificar o formato comprimido do draw.io, com **proteções**: limite de tamanho de entrada e de descompressão (contra *zip bomb*) e rejeição de `<!DOCTYPE>`/`<!ENTITY>` (contra XXE / *billion laughs*).
- Escrita em disco: apenas `--out` explícito nos extratores. `build_aws_icons.py` (ferramenta do mantenedor, não chamada pelo instalador nem pela skill) apaga e recria somente `assets/aws-icons/` da própria skill.
- Testei: `verify-motion.py --shipped` e `verify-geometry.py --all` → 103 arquivos, 0 achados; `self_check.py` detecta corretamente `<script>` injetado e imagem remota num HTML de teste.

## 3. HTML, SVG e JavaScript

- **817 SVG**: contêm apenas `svg`, `g`, `path`, `rect`, `polygon`, `title`, `defs`, `clipPath`. Nenhum `<script>`, `<foreignObject>`, `on*=` ou referência externa.
- **108 HTML**: nenhum `<iframe>`, `<object>`, `<embed>`, `<form>`, `meta refresh` ou URI `data:` com HTML/JS.
- **3 HTML com JavaScript** (`index.html`, `template-motion.html`, `example-policy-trace-animated.html`): código lido integralmente. Apenas troca de abas/iframe local e controle de animação (play/pause/teclado). Nenhum `fetch`, `XMLHttpRequest`, `WebSocket`, `eval`, `new Function`, cookies, `localStorage` ou `sendBeacon`.
- ⚠️ **Privacidade (baixo):** 98 dos 108 HTML (exemplos pré-gerados e `template-terminal.html`) incluem `<link>` para `fonts.googleapis.com`. Ao abri-los no navegador, seu IP é enviado ao Google. Não há execução de código. Os diagramas **novos** usam a Amazon Ember incluída e não precisam de CDN.

## 4. Binários

- 8 TTF e 8 WOFF2 com cabeçalhos válidos (`00 01 00 00` / `wOF2`), declarados como cópia sem modificação do pacote oficial da Amazon (`Amazon_Typefaces_Complete_Font_Set_Mar2020.zip`). Não comparei byte a byte com o zip oficial; se quiser essa garantia extra, baixe o zip da página da Amazon e compare os SHA-256.
- Alguns arquivos de fonte têm bit de execução (`chmod +x`) — inofensivo, apenas um descuido.
- 1 PDF (diretrizes de licença da fonte). Não é executado por nada no projeto.

## 5. Repositório e cadeia de suprimentos

- Sem submódulos, `.gitattributes`, Git LFS, symlinks versionados ou hooks Git ativos.
- Único caractere Unicode "invisível" encontrado: um BOM (`U+FEFF`) literal em `drawio_extract.py`, usado legitimamente para removê-lo da entrada. Sem caracteres bidi (ataque *Trojan Source*).
- Todos os commits são do mesmo autor (`masangbeom`).
- ⚠️ O README original manda instalar no Claude Code via `masangbeom/aws-diagram-design` — isso baixa do **upstream**, não do seu fork. Se você quer usar só o código revisado, instale a partir do seu fork ou de um clone local fixado no commit acima.

## 6. Instruções para o agente (prompt injection)

Revisei os 52 Markdown (que viram instruções do agente) procurando comandos de rede, instalação, exfiltração e instruções ocultas (incluindo comentários HTML). Nada suspeito: os comentários HTML são apenas anotações dentro de exemplos SVG.

- A skill pede que o agente execute os scripts Python locais acima (revisados).
- `export.md` instrui explicitamente a **não** instalar o Playwright automaticamente.
- `import-drawio.md` e `import-mermaid.md` tratam o conteúdo importado como **dados não confiáveis** (boa prática).
- ⚠️ `onboarding.md § URL` faz o agente acessar um site externo indicado por você para extrair cores e fontes. Use apenas sites confiáveis; conteúdo web pode tentar instruir o agente.
- O onboarding reescreve `references/style-guide.md` da própria skill (comportamento esperado, com aprovação).

## 7. Downloads externos que **você** pode decidir fazer

| Quando | Comando | Origem |
|---|---|---|
| Exportar PNG | `pip install playwright` | PyPI |
| Exportar PNG | `playwright install chromium` | CDN da Microsoft/Playwright (~150 MB) |

Recomendação: instale em um ambiente virtual dedicado (`python3 -m venv`) — veja [instalacao-kiro.md](instalacao-kiro.md).

## 8. Recomendações

1. Instale por **cópia** manual a partir de um commit fixado ([instalacao-kiro.md](instalacao-kiro.md)) e reavalie o diff a cada atualização (`git diff <commit-antigo>..<commit-novo>`).
2. Não abra os HTML de exemplo se o contato com o Google Fonts for um problema na sua rede.
3. Faça o onboarding por URL somente com sites confiáveis.
4. Licenças (questão jurídica, não de segurança): os ícones AWS e a Amazon Ember são redistribuídos sob termos próprios — leia `THIRD_PARTY_LICENSES.md` e o PDF de diretrizes antes de uso comercial.
