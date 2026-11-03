---
description: Delega uma tarefa ao agy (Gemini) e valida o que ele entregar antes de dar como pronta
argument-hint: "<o que o agy deve fazer>"
disable-model-invocation: true
---

Tarefa a delegar ao agy: $ARGUMENTS

1. Carregue a skill `agy` deste plugin e siga o que ela diz sobre delegar e validar.
2. Escreva um pedido autocontido num arquivo, no modelo de `references/prompt-modelo.md` da skill: caminhos absolutos, o que ler antes, o que fazer, o que não tocar e o critério de pronto.
3. Escolha o papel (`codigo`, `imagem`, `pesquisa`, `rapido`) e as pastas (`--dir`) que o agy pode usar. Rode `scripts/agy-tarefa` com o Bash em segundo plano.
4. Quando terminar, valide: compare o relato do agy com `alteracoes.txt`, leia o diff e rode as checagens do projeto (build, testes, tela).
5. Responda curto: o que o agy fez, com qual modelo, o que você conferiu e o que corrigiu.

Se a tarefa depende de muito contexto desta conversa, de segredos ou de uma ação irreversível, explique por que não vale delegar e ofereça fazer você mesmo.
