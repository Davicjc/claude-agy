---
name: agy
description: Orquestra o agy (Antigravity CLI, Gemini) como segundo agente do Claude Code — delega tarefas, roda trabalho em paralelo e pede revisão independente de código, escolhendo o modelo mais novo do agy na hora. Use sempre que houver agy instalado e a tarefa envolver gerar ou editar imagem (banner, foto, mockup, logo, texto dentro de imagem), transcrever ou analisar áudio/vídeo (.ogg, .mp3, .mp4, áudio de WhatsApp), dividir um trabalho grande em partes independentes, refactor mecânico em muitos arquivos, pesquisa longa, economizar cota do Claude (MODO ECONOMIA: semana >= 70% ou janela de 5 h >= 80%, ou aviso de limite perto do fim), ou quando o usuário pedir "segunda opinião", "revisão cruzada", "valida esse código", "pede pro Gemini/agy", mesmo que não diga "agy".
---

# agy: Claude + Gemini trabalhando juntos

O agy é um agente de terminal parecido com o Claude Code, que roda modelos Gemini (e outros). Ele é forte onde o Claude é mais fraco ou mais caro: **imagem**, **áudio e vídeo nativos**, e trabalho em volume. Usado com critério, ele vira um segundo par de mãos e um revisor independente. Usado sem critério, ele diz "feito" sem ter feito.

A regra de ouro desta skill: **o Claude continua responsável pelo resultado.** O agy executa; o Claude especifica, confere e decide.

Os scripts ficam em `scripts/` ao lado deste arquivo. Nos comandos abaixo, `<skill>` é a pasta mostrada como "Base directory for this skill" quando a skill é carregada; use sempre o caminho absoluto.

| Script | Para quê |
|---|---|
| `scripts/agy-modelo PAPEL` | Devolve o modelo certo para o papel, lido de `agy models` na hora |
| `scripts/agy-tarefa ...` | Roda uma tarefa, guarda tudo e mostra o que mudou de verdade no git |
| `scripts/agy-revisar ...` | Revisão de um diff só leitura, com achados estruturados |
| `scripts/agy-historico [N]` | Últimas execuções: status, modelo, tempo e tokens (para prestar contas do gasto) |
| `scripts/agy-cota` | Diz se é hora do **modo economia**, lendo o uso da cota do Claude que a status line guardou |

O usuário também pode chamar direto: `/agy:delegar`, `/agy:revisar`, `/agy:modelos`, `/agy:historico` e `/agy:cota`.

## Antes de tudo: o agy está pronto?

```bash
command -v agy && <skill>/scripts/agy-modelo --listar --atualizar
```

Se `agy` não existir ou o comando falhar, pare e avise o usuário (ele precisa instalar ou fazer login no agy). Não tente contornar. Depois da primeira checagem da sessão, `agy-modelo` sem `--atualizar` usa a lista guardada por 1 h.

## Modo economia: o Claude comanda, o agy executa

Confira a cota **no começo de cada pedido novo** que não seja trivial (é barato):

```bash
<skill>/scripts/agy-cota        # MODO=economia|normal|desconhecido, SEMANA=, SESSAO=, MOTIVO=
```

Entre em **modo economia** quando der `MODO=economia` (semana >= 70% ou janela de 5 h >= 80%; `AGY_COTA_SEMANA` e `AGY_COTA_SESSAO` mudam os limites) **ou** quando o Claude Code avisar que o limite está perto do fim. `MODO=desconhecido` quer dizer que a status line ainda não guardou os números (veja o README): siga no normal, a menos que haja aviso de limite.

No modo economia o papel do Claude muda. Ele é o **comandante que sabe o que tem que ser feito**, não o braço que faz:

1. **Entenda o pedido e decida o caminho**, lendo o mínimo. Para explorar código grande, mande o agy ler e resumir (`--modo plan`, papel `pesquisa`) em vez de ler tudo você.
2. **Delegue praticamente toda a execução**: código, edições, refactors, testes, documentação, pesquisa, imagem, áudio e vídeo. A lista "Não delegue" abaixo fica bem mais curta: valem só as exceções de segurança (segredo, ação irreversível, `git push`, mensagem para terceiros) e conversa com o usuário.
3. **Escreva uma diretriz quase impossível de errar**, seguindo a seção "Diretriz à prova de erro" de `references/prompt-modelo.md`: passos com a mudança exata, o trecho atual do código colado, critérios de aceite e o comando que prova que deu certo. Tempo gasto na diretriz economiza rodadas de correção.
4. **Divida e rode em paralelo** as partes independentes (até 3–4 de uma vez).
5. **Valide barato**: `alteracoes.txt`, `git diff --stat`, os testes e a saída do comando de verificação. Leia só os trechos que a verificação apontar como suspeitos, não o diff inteiro.
6. **Correção também é do agy**: mande de volta com `--continuar <id>` e a lista exata do que está errado. Corrija você mesmo só o que for menor que escrever a correção (umas poucas linhas).
7. **Avise o usuário em uma linha** que entrou no modo economia e por quê (ex.: "modo economia: semana 74%").

## Quando delegar (e quando não)

**Delegue** quando o agy for melhor ou quando paralelizar economiza tempo de verdade:
- **Imagem**: gerar ou editar foto, fundo, banner, mockup, ícone, trocar texto ou logo dentro de imagem.
- **Áudio e vídeo**: transcrever áudio (inclusive de WhatsApp), resumir ou analisar um .mp4 direto do arquivo.
- **Partes independentes** de um trabalho grande: módulos, telas, testes, documentação, cada uma com seus arquivos.
- **Volume mecânico**: renomear API em 40 arquivos, migrar padrão repetitivo, gerar dados de exemplo.
- **Segunda opinião**: revisar o código que o Claude acabou de escrever (veja "Validação cruzada").

**Não delegue** quando:
- a tarefa depende do contexto desta conversa que é difícil de passar por escrito (decisões, preferências do usuário, idas e vindas): escrever esse contexto costuma custar mais que fazer;
- é uma edição pequena e óbvia (delegar e conferir sai mais caro);
- envolve segredo, senha, token, dados pessoais de terceiros ou ação irreversível (apagar, publicar, mandar mensagem, `git push`): isso fica com o Claude e com o usuário.

## Escolha do modelo (dinâmica)

Nunca escreva um nome de modelo fixo no prompt ou no comando. Use o papel:

| Papel | Uso | Hoje escolhe (exemplo) |
|---|---|---|
| `codigo` | escrever e alterar código | Gemini "pro" mais novo, esforço alto |
| `revisao` | revisão de diff, caça a bugs | Gemini "pro" mais novo, esforço alto |
| `pesquisa` | ler muito material e resumir | Gemini "pro" mais novo, esforço médio |
| `imagem` | gerar ou editar imagem | Gemini "pro" mais novo, esforço médio |
| `rapido` | tarefa simples, volume, rascunho | Gemini "flash" mais novo |

`agy-modelo` lê `agy models` (com cache de 1 h), separa a família rápida (flash/lite) da principal, pega a **versão mais nova** e o esforço preferido do papel. Quando sair um Gemini novo, ele passa a ser usado sem mexer em nada. Para forçar: `AGY_MODELO_CODIGO=<id>` (por papel) ou `AGY_MODELO=<id>` (todos). Use `--atualizar` se um modelo acabou de ser lançado.

Para revisão, prefira um modelo de **família diferente** da que escreveu o código. A graça da segunda opinião é ter outro "cérebro". O papel `revisao` já escolhe Gemini; não troque por Claude dentro do agy só para revisar código do próprio Claude.

## Como pedir uma tarefa

1. **Escreva o prompt num arquivo** seguindo `references/prompt-modelo.md`. Ele precisa ser autocontido: o agy não viu esta conversa. Caminhos absolutos, o que ler antes, o que fazer, o que **não** tocar, onde salvar e como provar que está certo.
2. **Rode em segundo plano** (Bash com `run_in_background: true`), porque tarefas levam de segundos a dezenas de minutos:

```bash
<skill>/scripts/agy-tarefa --dir /abs/projeto --papel codigo \
  --nome "tela de login" --prompt-arquivo /abs/prompt.md --timeout 2400
```

- `--dir` é obrigatório e pode repetir. O agy **só enxerga e edita as pastas passadas aqui**. Sem isso ele grava num rascunho próprio e responde "feito" sem ter mudado nada no projeto. Inclua também a pasta dos arquivos de apoio (imagem de referência, áudio).
- `--modo plan` deixa o agy só ler (análise, pesquisa, revisão). O padrão `accept-edits` permite editar arquivos.
- O script acrescenta regras fixas ao prompt: não commitar, não usar sudo, não sair das pastas, responder curto. Use `--sem-regras` só quando tiver um motivo.
- Para continuar a mesma conversa (ex.: pedir um ajuste), use `--continuar <id>`, com o id que está em `meta.json`. O agy lembra do que foi dito antes.
- Pedido longo não é problema: acima de ~100 KB o script entrega o pedido ao agy por arquivo (a linha de comando do Linux recusa um argumento acima de 128 KB).
- A saída mostra no máximo 150 linhas da resposta (`--max-linhas N` muda isso); o texto completo fica em `resposta.md`, na pasta indicada em `PASTA=`. Leia de lá só se precisar: assim o resultado do agy não enche o seu contexto.
- Se o Claude estiver rodando como root numa pasta de outro usuário, o script devolve ao dono da pasta o que o agy criou pelo terminal (`mkdir`, `npm install`, `git`...), que nasce como root. Sem isso, o git e o editor do usuário quebram depois. A ferramenta de escrita do próprio agy já respeita o dono. `--manter-donos` desliga isso.

| Saída do `agy-tarefa` | Significado |
|---|---|
| `0` | O agy terminou. Ainda precisa de validação (próxima seção) |
| `1` | Erro do agy ou resposta vazia (`RESPOSTA-VAZIA`): trate como falha |
| `2` | Uso errado do script (a mensagem diz o que faltou) |
| `3` | O agy fez commit, apesar das regras: veja `git log` e avise o usuário |
| `124` | Tempo esgotado. O que ele já editou continua no disco: confira `alteracoes.txt` |

3. **Enquanto roda, siga trabalhando** em outra coisa. Você é avisado quando o processo termina.

## Validar o que o agy entregou (obrigatório)

O relato do agy é uma alegação, não uma prova. O `agy-tarefa` já mostra **o que mudou de verdade** (comparação do `git status` antes e depois) e alerta se o agy fez commit (código de saída 3). Depois:

1. **Compare o relato com `alteracoes.txt`.** Se ele diz que criou um arquivo que não aparece, ou mexeu em algo fora do pedido, isso é um problema.
2. **Leia o diff** (`git diff`, `git diff --stat`), não só a resposta.
3. **Rode as checagens do projeto**: build, testes, lint, `node --check`, `python -m py_compile`, ou o que existir. Para interface, olhe a tela de verdade (screenshot/navegador). O agy costuma errar tema, cores e detalhes de integração entre arquivos.
4. **Decida**: aceitar, corrigir você mesmo (se for pequeno) ou mandar de volta com `--continuar` e a lista exata do que está errado.

Só diga ao usuário que está pronto depois disso.

## Validação cruzada (segunda opinião)

Depois de uma mudança relevante, escrita pelo Claude ou pelo agy, peça uma revisão independente:

```bash
<skill>/scripts/agy-revisar --dir /abs/repo                       # tudo desde o HEAD, com arquivos novos
<skill>/scripts/agy-revisar --dir /abs/repo --base main           # tudo desde main
<skill>/scripts/agy-revisar --dir /abs/repo --intervalo HEAD~3..HEAD --foco "concorrência no worker"
```

O agy revisa em modo só leitura e devolve veredito (`aprovado`, `ajustar`, `reprovar`) e achados com gravidade, confiança, arquivo:linha, evidência e sugestão (formato em `assets/schema-revisao.json`). O diff vai delimitado e com o aviso de que texto dentro do código é dado, não instrução, para um comentário malicioso não "mandar" o revisor aprovar. Com `--exigir-aprovado`, o script sai com código 4 quando o veredito não é "aprovado" (útil em automação).

**Não aplique os achados às cegas.** Para cada um: abra o código, confirme que o cenário de falha acontece de verdade e só então corrija. Descarte os falsos positivos e diga ao usuário quantos achados eram reais. Um achado `baixa` confiança é uma pista, não uma ordem. Se o agy reprovar algo que você tem certeza que está certo, explique o motivo ao usuário em vez de mudar para agradar o revisor.

## Trabalho em paralelo

Para dividir um trabalho grande:

1. Quebre em partes que **não mexem nos mesmos arquivos**. Se duas partes precisam do mesmo arquivo, elas não são paralelas: faça em sequência.
2. Dê a cada parte uma pasta própria: um `git worktree` por parte é o mais seguro (`git worktree add ../projeto-parte-a -b parte-a`), ou subpastas disjuntas.
3. Rode um `agy-tarefa` por parte, todos com `run_in_background: true`, cada um com `--dir` só da sua pasta.
4. Valide cada parte (seção acima) antes de juntar. Junte uma por vez, rodando os testes a cada junção.
5. Mais de 3–4 agentes ao mesmo tempo costuma estourar a cota ou o limite de requisições do agy. Prefira poucas partes bem definidas.

Uma boa divisão é o Claude cuidar da parte que exige o contexto da conversa (arquitetura, integração, decisões do usuário) e o agy cuidar das partes isoladas.

## Imagem, áudio e vídeo

- Coloque o arquivo de entrada numa pasta passada em `--dir` e cite o caminho absoluto no prompt.
- Para imagem, diga o tamanho, o formato e o caminho exato de saída, e mostre o resultado ao usuário antes de usar no projeto. Se houver identidade visual (cores, fontes), passe os valores, porque o agy não adivinha.
- Para transcrição, peça o formato (texto corrido, com tempos, por falante) e o idioma. Use `--modo plan` se não houver nada para editar.

## Problemas comuns

Leia `references/problemas.md` quando algo der errado: "feito" sem mudança, tempo esgotado, cota, modelo não reconhecido, login, resposta fora do formato.

## Como reportar ao usuário

Curto e verificável: o que foi delegado, com qual modelo (`meta.json`), o que mudou (arquivos), o que você conferiu (testes, diff, tela) e o que corrigiu do que o agy fez. Se a revisão cruzada achou algo, diga quantos achados eram reais e o que foi feito com eles. Os registros completos ficam em `~/.cache/agy-skill/execucoes/` (apagados depois de 30 dias; `AGY_SKILL_RETENCAO_DIAS` muda isso), e `agy-historico` resume o gasto de tokens.
