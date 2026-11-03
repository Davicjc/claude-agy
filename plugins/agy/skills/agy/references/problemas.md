# Problemas comuns com o agy

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Responde "feito", mas `alteracoes.txt` mostra "nenhuma alteração" | Faltou `--dir` da pasta certa, ou o arquivo está fora das pastas passadas | Rode de novo com `--dir` apontando para a pasta que contém os arquivos. O `cd` do terminal não define o workspace do agy |
| `alteracoes.txt` diz "não é repositório git" | A pasta não tem git | Confira os arquivos à mão (`ls -la`, `stat`), ou faça `git init` numa cópia antes de delegar |
| Código de saída 3 / "ALERTA: o HEAD mudou" | O agy fez commit ou trocou de branch, apesar da regra | Veja `git log -3` e `git reflog`; desfaça com cuidado e avise o usuário. Não faça push |
| Código de saída 124 / `TEMPO-ESGOTADO` | Tempo esgotado. O agy devolve `SUCCESS` com `finished_reason: print_timeout` e resposta vazia ou parcial; o script converte isso em 124 | Divida a tarefa ou aumente `--timeout`. O que já foi editado continua no disco: confira `alteracoes.txt` |
| `RESPOSTA-VAZIA` (código 1) | O agy disse sucesso, mas não escreveu nada | Trate como falha: veja `stderr.log` e repita, de preferência com o prompt mais claro |
| `status: ERROR` com "not recognized as a known model" | Modelo removido ou id errado | `agy-modelo --listar --atualizar` e confira as variáveis `AGY_MODELO*` |
| Erro de cota ou limite de requisições (429, "quota") | Muitas chamadas seguidas ou em paralelo | Espere alguns minutos, reduza o paralelismo ou use `--papel rapido` |
| `agy models` vazio ou erro de autenticação | agy deslogado | Peça ao usuário para rodar `agy` interativo uma vez e fazer login |
| Revisão sem `estruturado.json` | O modelo não seguiu o schema ou a resposta foi cortada | Repita com `--papel revisao` (esforço alto) ou diminua o diff (`--intervalo`) |
| A resposta fala de arquivos com `file:///` | Normal: o agy gera links locais | Leia os arquivos pelo caminho normal |
| Pede permissão e trava | Modo interativo acidental | Os scripts usam `-p` (sem perguntas). Não use `--dangerously-skip-permissions` |

Os registros de cada execução ficam em `~/.cache/agy-skill/execucoes/<data>-<nome>/` (`prompt.md`, `resultado.json`, `resposta.md`, `alteracoes.txt`, `stderr.log`, `meta.json`). Comece por `stderr.log` e `resultado.json`.
