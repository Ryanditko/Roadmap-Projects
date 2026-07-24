# Aplicativo de Quiz

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um quiz de múltipla escolha que conduz o usuário por um conjunto de perguntas, uma de cada vez, registra suas respostas e mostra uma tela de resultados pontuada ao final. As perguntas vivem em um array local, então o trabalho é todo sobre fluxo e estado: em qual pergunta estamos, o que foi respondido e o que a tela mostra agora. É uma introdução limpa à renderização condicional e a uma pequena máquina de estados — início, em andamento e finalizado — além da renderização acessível de escolhas em grupo de rádio, onde usuários de teclado podem se mover e selecionar sem um mouse.

## Pré-requisitos

- Fundamentos de HTML, CSS e JavaScript
- Iteração de arrays e rastreamento de índice
- Renderização condicional (mostrar uma view com base no estado)
- Grupos de rádio e botões acessíveis

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar o progresso do quiz como estado explícito (índice atual, respostas, fase)
- Renderizar uma pergunta por vez e avançar/voltar pelo conjunto
- Calcular uma pontuação a partir das respostas registradas, não do DOM
- Mostrar uma tela de resultados com uma revisão por pergunta
- Construir um grupo de rádio acessível com seleção por teclado e foco claro

## Requisitos Funcionais

1. As perguntas são mostradas uma de cada vez com suas opções de resposta.
2. O usuário seleciona uma resposta por pergunta e pode ir para a próxima pergunta.
3. Um indicador de progresso mostra a posição atual (ex.: "Pergunta 3 de 10").
4. Ao final, uma tela de resultados mostra a pontuação e quais respostas estavam certas ou erradas.
5. As opções de resposta formam um grupo de rádio acessível navegável por teclado.
6. O usuário não pode avançar além da última pergunta sem ver os resultados.
7. Uma opção de reiniciar reseta todo o estado e retorna à primeira pergunta.

## Marcos Sugeridos

1. **Marco 1 — Renderizar e responder:** Mostre uma pergunta com opções selecionáveis e registre a escolha.
2. **Marco 2 — Navegação e progresso:** Mova entre perguntas e exiba o progresso.
3. **Marco 3 — Pontuação e revisão:** Calcule a pontuação e renderize uma tela de resultados/revisão.

## Esboço de Dados e Interface

```text
Question
  id:       string
  prompt:   string
  options:  string[]
  answer:   number   (índice da opção correta)

Estado do quiz
  phase:    "start" | "playing" | "finished"
  index:    number
  answers:  (number | null)[]   (uma posição por pergunta)

Layout (playing)
+------------------------------------------+
| Pergunta 3 de 10        [====------]     |
+------------------------------------------+
| O que significa CSS?                     |
|  ( ) Cascading Style Sheets              |
|  ( ) Computer Style System               |
|  ( ) Creative Styling Syntax             |
+------------------------------------------+
| [ Voltar ]                 [ Próxima ]   |
+------------------------------------------+
```

## Desafios Extras

- Adicione um cronômetro opcional de contagem regressiva por pergunta que avança automaticamente ao expirar.
- Randomize a ordem das perguntas e das opções a cada execução.
- Suporte múltiplos quizzes/categorias escolhidos em uma tela inicial.
- Persista uma pontuação máxima local entre sessões com `localStorage`.

## Definição de Pronto

- [ ] Exatamente uma pergunta é mostrada por vez com suas opções.
- [ ] O progresso reflete com precisão a pergunta atual.
- [ ] A pontuação final corresponde às respostas registradas.
- [ ] As opções são um grupo de rádio navegável por teclado com foco visível.
- [ ] Reiniciar reseta totalmente o estado de volta à primeira pergunta.

## Armadilhas Comuns

- Ler a resposta selecionada do DOM em vez de rastreá-la no estado.
- Erros de "um a mais/menos" no contador de progresso ou no limite da última pergunta.
- Usar `<div>`s clicáveis para opções, quebrando o uso por teclado e leitor de tela.
- Recalcular a pontuação incorretamente quando um usuário muda uma resposta anterior.
- Não resetar cada parte do estado ao reiniciar, vazando a execução anterior.

## Recursos

- [MDN: <input type="radio">](https://developer.mozilla.org/pt-BR/docs/Web/HTML/Element/input/radio) — agrupamento e comportamento de teclado padrão.
- [WAI-ARIA: Radio Group pattern](https://www.w3.org/WAI/ARIA/apg/patterns/radio/) — grupos de rádio personalizados acessíveis.
- [MDN: Condicionais em JavaScript](https://developer.mozilla.org/pt-BR/docs/Learn/JavaScript/Building_blocks/conditionals) — alternar views por estado.
- [web.dev: Learn Accessibility — Focus](https://web.dev/learn/accessibility/focus) — gerenciar e mostrar o foco.
