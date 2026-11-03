# claude-agy

**Português** | **[Read in English](README.en.md)**

Plugin do **Claude Code** que faz o Claude e o **agy** (Antigravity CLI, com modelos Gemini) trabalharem juntos.
O Claude especifica, delega, confere e decide. O agy executa onde é mais forte (imagem, áudio, vídeo e
trabalho em volume) e serve de **revisor independente** do código.

<img src="plugins/agy/.claude-plugin/icon.png" width="96" alt="ícone do plugin">

---

## O que ele faz

| Recurso | Como |
|---|---|
| **Delegar tarefas** | `agy-tarefa` roda o pedido no agy em segundo plano, com as pastas liberadas, tempo limite e regras de segurança fixas no prompt |
| **Não acreditar no "feito"** | Compara o `git status` antes e depois e lista **o que mudou de verdade**. Avisa (código de saída 3) se o agy fizer commit |
| **Modelo sempre atual** | `agy-modelo` lê `agy models` na hora e escolhe a versão mais nova para cada papel (código, revisão, imagem, pesquisa, rápido). Gemini novo lançado = usado sem editar nada |
| **Revisão cruzada** | `agy-revisar` manda um `git diff` para o agy revisar **só lendo** e recebe veredito e achados num formato fixo: gravidade, confiança, arquivo:linha, evidência e sugestão |
| **Trabalho em paralelo** | A skill ensina o Claude a dividir em partes sem arquivos em comum (um `git worktree` por parte), rodar várias ao mesmo tempo e validar cada uma antes de juntar |
| **Imagem, áudio e vídeo** | Banner, mockup, ícone, transcrição de áudio de WhatsApp, análise de .mp4, direto do arquivo |
| **Tudo registrado** | Cada execução guarda prompt, resposta, JSON, tokens, modelo e alterações em `~/.cache/agy-skill/execucoes/` |

O Claude usa a skill sozinho quando a tarefa pede (imagem, áudio, trabalho grande, "pede uma segunda opinião"). Você também pode pedir direto, por exemplo:

- *"Pede pro agy gerar o banner do README com as cores da marca."*
- *"Transcreve esse áudio do WhatsApp que tá em ~/Downloads."*
- *"Divide essa migração em partes e roda em paralelo com o agy."*
- *"Faz uma revisão cruzada do que mudou desde a main."*

---

## Testado de verdade

| Teste | Resultado |
|---|---|
| Corrigir um bug e criar teste (`agy-tarefa`, modelo rápido) | Feito em 27 s; o relatório do git confirmou os arquivos alterados e criados |
| Revisão de um diff com 3 problemas plantados (injeção de SQL, senha no código, divisão por zero), um deles em arquivo novo não rastreado | Os **3 encontrados**, cada um com cenário de falha concreto e sugestão; veredito "reprovar" |
| Pedir de propósito um commit ao agy | Detectado: código de saída 3, com o commit e os arquivos listados |
| Esquecer `--dir` | Recusado antes de chamar o agy |
| Simular modelos que ainda não existem (`gemini-5-krypton-*`, `gemini-4.5-flash-*`) | Escolheu os mais novos sozinho |
| O ícone deste plugin | Gerado pelo próprio agy com `agy-tarefa --papel imagem` |
| **A skill revisando a si mesma** (`agy-revisar` nos próprios scripts) | 6 achados, **todos confirmados e corrigidos**: loop infinito com opção sem valor, erro do `git diff` engolido, edição em arquivo já modificado passando despercebida, cache velho escondendo agy deslogado, pastas iguais em paralelo e tempo esgotado aparecendo como sucesso |

---

## Instalação

### Requisitos
- [Claude Code](https://claude.com/claude-code)
- **agy** (Antigravity CLI) instalado e logado: `agy models` precisa listar os modelos
- `git`, `python3` e `bash` (padrão em Linux e macOS)

### No Claude Code

Em um passo (Claude Code 2.1.275 ou mais novo):

```
/plugin install agy --marketplace Davicjc/claude-agy
```

Ou em dois passos (qualquer versão):

```
/plugin marketplace add Davicjc/claude-agy
/plugin install agy@davicjc
```

Depois rode `/reload-plugins` (ou abra uma sessão nova). Para conferir: peça *"lista os modelos do agy"*.

### Atualizar

```
/plugin marketplace update davicjc
```

---

## Os scripts (também funcionam sozinhos)

Ficam em `plugins/agy/skills/agy/scripts/`.

```bash
# Qual modelo usar (lido do agy agora)
agy-modelo --listar
agy-modelo codigo                      # -> ex.: gemini-4-argon-high

# Rodar uma tarefa que edita arquivos
agy-tarefa --dir ~/projeto --papel codigo --nome "tela de login" \
           --prompt-arquivo pedido.md --timeout 2400

# Só ler/analisar (sem editar)
agy-tarefa --dir ~/projeto --modo plan --papel pesquisa --prompt "Explique como o cache funciona"

# Revisão cruzada
agy-revisar --dir ~/projeto                      # tudo desde o HEAD, com arquivos novos
agy-revisar --dir ~/projeto --base main          # desde a main
agy-revisar --dir ~/projeto --intervalo HEAD~3..HEAD --foco "concorrência"
```

**Forçar um modelo:** `AGY_MODELO_CODIGO=<id>` (por papel) ou `AGY_MODELO=<id>` (todos).

**Códigos de saída do `agy-tarefa`:** `0` ok · `1` erro do agy ou resposta vazia · `2` uso errado · `3` ok, mas o agy fez commit · `124` tempo esgotado.

---

## Regras de segurança embutidas

- O agy só enxerga e edita as pastas passadas em `--dir` (sem `--dir`, o script recusa).
- Todo pedido leva regras fixas: não commitar, não dar push, não apagar trabalho, não usar `sudo`, não sair das pastas e parar e explicar quando algo impedir a tarefa.
- Nunca usa `--dangerously-skip-permissions`.
- A revisão roda em modo `plan` (só leitura).
- A skill orienta o Claude a **conferir tudo** (diff, testes, tela) antes de dizer que está pronto, e a não aplicar achados de revisão sem confirmar cada um.

Problemas comuns e soluções: [`plugins/agy/skills/agy/references/problemas.md`](plugins/agy/skills/agy/references/problemas.md).

---

## Estrutura

```
.claude-plugin/marketplace.json          marketplace "davicjc"
plugins/agy/
  .claude-plugin/plugin.json, icon.png
  skills/agy/
    SKILL.md                             quando e como delegar, validar e paralelizar
    scripts/agy-modelo                   escolha dinâmica de modelo
    scripts/agy-tarefa                   execução rastreável
    scripts/agy-revisar                  revisão cruzada estruturada
    assets/schema-revisao.json           formato dos achados
    references/prompt-modelo.md          como escrever o pedido
    references/problemas.md              solução de problemas
```

---

## Licença

MIT. Feito por [Davi Castro](https://davicjc.com).
