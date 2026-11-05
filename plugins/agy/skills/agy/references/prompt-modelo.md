# Modelo de prompt para o agy

O agy não viu a conversa com o usuário. Tudo que ele precisa saber tem que estar escrito. Um bom prompt deixa pouca margem para ele adivinhar, e cada adivinhação é um lugar onde ele pode errar.

## Estrutura

```markdown
# Tarefa: <uma frase com o resultado esperado>

## Contexto
- Projeto: <o que é, em 1–2 frases> em /abs/caminho/projeto
- Stack: <linguagem, framework, versões que importam>
- Por que esta tarefa existe: <o problema que resolve>

## Leia antes de começar
- /abs/caminho/arquivo-principal.ext (como o código atual faz X)
- /abs/caminho/exemplo-parecido.ext (siga este padrão)

## O que fazer
1. <passo concreto>
2. <passo concreto>

## Não faça
- Não mexa em <arquivos/pastas>, que são de outra parte do trabalho.
- Não mude <API/contrato/estilo> sem necessidade.
- Não adicione dependências novas.

## Pronto quando
- <critério verificável, ex.: `npm test` passa; a função X retorna Y para Z>
- Rode <comando de verificação> e informe o resultado.

## Resposta
Curta: o que fez, os arquivos criados ou alterados e o resultado da verificação.
```

## Dicas que fazem diferença

- **Mostre o padrão a seguir.** "Siga o mesmo formato de `modules/clientes.js`" funciona melhor que descrever o padrão em palavras.
- **Dê valores, não adjetivos.** "Azul #1E40AF, fonte Inter 14px" em vez de "azul bonito".
- **Defina o fim.** Sem critério de pronto, o agy para quando acha que parou. Dê um comando para rodar.
- **Separe o que é seu do que é dele.** Em paralelo, liste os arquivos que outra parte está mexendo.
- **Uma tarefa por chamada.** Se a lista de passos passa de ~8 itens ou mistura assuntos, divida.

## Diretriz à prova de erro (modo economia)

No modo economia o agy faz quase tudo, então cada diretriz precisa sair certa de primeira. Além da estrutura acima:

- **Cole o estado atual.** O trecho do código que vai mudar, com o caminho e as linhas, em vez de "abra o arquivo e procure". Ele erra menos quando vê o que existe.
- **Diga a mudança exata.** "Na função `salvar()` de `/abs/app/db.js`, troque `set` por `update` e trate o erro `not-found` criando o documento", não "melhore o salvamento".
- **Uma decisão por linha, e já tomada.** Tudo que dependia de escolha (nome, biblioteca, formato, texto da interface) vai decidido. Nada de "use o que achar melhor".
- **Critérios de aceite verificáveis**, e o comando exato que prova cada um, com a saída esperada. Peça para colar a saída na resposta.
- **Limites explícitos**: os arquivos que pode tocar (o resto é proibido), sem dependência nova, sem mudar contrato ou estilo.
- **Saída para dúvida.** "Se algo aqui não bater com o código (arquivo não existe, função com outro nome, teste já quebrado antes), pare e responda `BLOQUEADO: <motivo>`, sem improvisar." Melhor parar que inventar.
- **Pequena o bastante para acertar.** Se passou de ~8 passos ou mistura assuntos, divida em tarefas e rode em sequência ou em paralelo.

Antes de mandar, releia como se fosse o agy, sem ter visto a conversa: se em algum ponto você teria que adivinhar, a diretriz ainda não está pronta.

## Exemplo curto (imagem)

```markdown
# Tarefa: criar o banner do README do projeto Ecolyra

## Contexto
- Projeto: app de letras traduzidas em /home/u/proj/ecolyra
- Identidade: fundo #0B0F14, destaque #4ADE80, fonte IBM Plex Sans

## Leia antes
- /home/u/proj/ecolyra/brand/logo.svg (use este logo, sem alterar)

## O que fazer
Banner 1280x640 PNG com o logo à esquerda e a frase "Letras traduzidas enquanto a música toca" à direita.

## Não faça
- Não use degradê, brilho ou efeitos de vidro.

## Pronto quando
- Arquivo salvo em /home/u/proj/ecolyra/.github/readme/banner.png, 1280x640.
```
