# Histórico de versões

## 1.2.0 — 04/11/2026

**Modo economia**
- Com a cota do Claude alta (semana ≥ 70% ou janela de 5 h ≥ 80%), a skill manda o Claude virar só o comandante: entender, decidir, escrever uma diretriz quase impossível de errar e delegar ao agy praticamente toda a execução, inclusive as correções (`--continuar`). As exceções de segurança continuam com o Claude.
- `agy-cota`: lê o uso da cota que a status line guardou e responde `MODO=economia|normal|desconhecido`. Limites em `AGY_COTA_SEMANA` e `AGY_COTA_SESSAO`. `agy-cota --statusline` é a própria status line: guarda os números e mostra `Opus · 5h 34% · semana 74% · modo agy`.
- Comando `/agy:cota`.
- Testes: 114 verificações (98 como usuário comum e 16 como root); 19 novas para o `agy-cota`.
- `references/prompt-modelo.md`: seção "Diretriz à prova de erro" (estado atual colado, mudança exata, decisões já tomadas, aceite com comando, saída `BLOQUEADO:` em vez de improvisar).

## 1.1.1 — 03/11/2026

- `agy-historico`: concordância ("1 execução" em vez de "1 execuções").

## 1.1.0 — 03/11/2026

**Correções**
- Pedido ou diff acima de 128 KB quebrava com "Argument list too long" (limite de um argumento no Linux). Agora pedidos acima de ~100 KB vão ao agy por arquivo.
- Não funcionava no macOS de fábrica: usava recursos do bash 4 e o `timeout` do GNU. Agora roda no bash 3.2 e sem `timeout` (usa o limite do próprio agy; com `gtimeout`, usa os dois).
- Rodando como root (ex.: `sudo claude`), os arquivos que o agy criava pelo terminal ficavam com dono root e o git do usuário quebrava depois. Agora voltam para o dono da pasta, só se tiverem um único link e estiverem no mesmo disco (um hardlink não serve para entregar um arquivo do root ao usuário). `--manter-donos` desliga.
- O próprio `git status` dos scripts, rodado como root, regravava o `.git/index` com dono root. Agora nenhum git dos scripts grava no repositório (`GIT_OPTIONAL_LOCKS=0`).
- Quando o git recusa um repositório de outro usuário (root sem `sudo`), os scripts explicam o motivo em vez de dizer que a pasta não é git. A proteção `safe.directory` não é desligada: ela impede que um repositório malicioso execute comandos como root.
- `agy-revisar` numa subpasta passa a revisar a partir da raiz do repositório (antes, os caminhos do diff e dos arquivos novos não batiam).

**Novidades**
- Comandos `/agy:delegar`, `/agy:revisar`, `/agy:modelos` e `/agy:historico`.
- `agy-historico`: últimas execuções com status, modelo, tempo e tokens, e o total por modelo.
- `agy-revisar --exigir-aprovado`: sai com código 4 quando o veredito não é "aprovado", para automação.
- Revisão protegida contra instruções escondidas no código: o diff vai delimitado e marcado como dado.
- A resposta mostrada ao Claude é limitada a 150 linhas (`--max-linhas`); o texto completo fica em `resposta.md`. Economiza o contexto do Claude.
- Uma só chamada de `git hash-object` para todos os arquivos, em vez de uma por arquivo.
- Execuções com mais de 30 dias são apagadas sozinhas (`AGY_SKILL_RETENCAO_DIAS`).
- Testes automáticos com um agy falso: 94 verificações (78 como usuário comum, no Linux e no macOS com bash 3.2, e 16 como root, no Linux), mais ShellCheck, no GitHub Actions.
- A revisão cruzada desta versão pelo próprio agy achou 4 problemas; 3 corrigidos como sugerido e 1 de outro jeito (a sugestão de desligar o `safe.directory` como root foi recusada por segurança). A quarta rodada confirmou as correções e achou 1 detalhe de ordem, também corrigido.

## 1.0.0 — 03/11/2026

Primeira versão: `agy-modelo`, `agy-tarefa`, `agy-revisar` e a skill `agy`. Testada com o agy real; a revisão cruzada da própria skill achou 6 problemas, todos corrigidos antes de publicar.
