---
description: Revisão cruzada do agy (Gemini) sobre as mudanças atuais do repositório, com cada achado conferido
argument-hint: "[--base main | --intervalo A..B] [--foco \"texto\"]"
disable-model-invocation: true
---

Faça uma revisão cruzada, com o agy, das mudanças do repositório em que estamos trabalhando.

1. Carregue a skill `agy` deste plugin: ela informa a pasta dos scripts e explica como tratar os achados.
2. Descubra a raiz do repositório (`git rev-parse --show-toplevel`) e rode, com o Bash em segundo plano:
   `<pasta da skill>/scripts/agy-revisar --dir <raiz> $ARGUMENTS`
3. Quando terminar, para **cada** achado: abra o código e confirme se o cenário de falha acontece de verdade. Classifique como real ou falso positivo, com o motivo.
4. Responda com uma tabela curta (achado, gravidade, real ou falso, motivo) e o veredito do agy. Não corrija nada sem o usuário pedir: ofereça corrigir os achados reais.

Se não houver mudanças, diga isso e sugira `--base main` ou `--intervalo`.
