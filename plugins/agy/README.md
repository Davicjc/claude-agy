# agy: Claude + Gemini trabalhando juntos

Plugin do Claude Code que orquestra o agy (Antigravity CLI): o Claude atua como Tech Lead e arquiteto (cuida da "barra pesada"), enquanto o agy (Gemini) executa o trabalho braçal de volume, multimídia (imagem, áudio, vídeo) e revisão cruzada independente — economizando tokens e mantendo o contexto do Claude limpo.

Delega com pastas liberadas e regras de segurança, audita o que mudou de verdade no git, escolhe o modelo Gemini mais novo na hora (`agy models`) e faz revisão cruzada de código em modo só leitura com achados estruturados. Comandos: `/agy:delegar`, `/agy:revisar`, `/agy:modelos` e `/agy:historico`.

Instalar: `/plugin install agy --marketplace Davicjc/claude-agy`

Requisitos: agy instalado e logado, `git`, `python3` e `bash`.

Documentação completa, instalação e testes: [README do repositório](https://github.com/Davicjc/claude-agy#readme).
