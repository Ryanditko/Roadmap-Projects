# Calculadora (UI)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa uma calculadora funcional com teclado, visor e as quatro operações básicas. O layout visual é a metade fácil; a lição real é modelar o comportamento da calculadora como uma pequena máquina de estados. Um usuário toca `7`, `+`, `3`, `=` — cada tecla significa algo diferente dependendo do que veio antes. Você vai rastrear a entrada atual, o operador pendente e o valor acumulado, e decidir o que cada botão faz em cada estado. Acerte esse modelo e casos limítrofes como encadear operações ou pressionar `=` duas vezes deixam de ser surpresas.

## Pré-requisitos

- HTML, CSS e JavaScript básicos
- CSS Grid ou Flexbox para dispor um teclado de botões
- Conforto com lógica de `switch`/condicionais e parsing de números
- Um editor de código e um navegador com ferramentas de desenvolvedor

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Representar o comportamento interativo como estado explícito em vez de ler o visor de volta como dado
- Tratar a ordem das operações de uma calculadora simples da esquerda para a direita e operações encadeadas
- Formatar a saída numérica e evitar artefatos brutos de ponto flutuante
- Suportar entrada por mouse/toque e por teclado físico para as mesmas ações
- Proteger contra estados inválidos como múltiplos decimais ou divisão por zero

## Requisitos Funcionais

1. A calculadora realiza adição, subtração, multiplicação e divisão.
2. O visor mostra o número sendo digitado e o resultado após `=`.
3. Pressionar um operador após um resultado usa esse resultado como novo operando à esquerda (encadeamento).
4. Um botão limpar reseta todo o estado; um backspace remove o último dígito digitado.
5. Apenas um ponto decimal é permitido por número.
6. Divisão por zero mostra um estado de erro claro em vez de `Infinity` ou `NaN`.
7. Teclas de número e operador no teclado físico espelham os botões da tela.

## Marcos Sugeridos

1. **Marco 1 — Entrada e visor:** Construa o teclado e mostre dígitos, decimais e backspace.
2. **Marco 2 — Operação única:** Armazene um operador e o operando à esquerda, calcule no `=`.
3. **Marco 3 — Encadeamento e proteções:** Encadeie operações, trate limpar, decimais e divisão por zero.

## Esboço de Dados e Interface

```text
Estado
  current:   string    (dígitos sendo digitados, ex.: "12.5")
  operator:  "+"|"-"|"*"|"/"|null
  accumulator: number|null
  justEvaluated: boolean

Layout (grade de 4 colunas)
+-------------------------------+
|            123.45             |  <- visor
+-------------------------------+
|  C  | +/- |  %  |   /         |
|  7  |  8  |  9  |   *         |
|  4  |  5  |  6  |   -         |
|  1  |  2  |  3  |   +         |
|    0      |  .  |   =         |
+-------------------------------+
```

## Desafios Extras

- Adicione um histórico/fita das cálculos anteriores.
- Adicione teclas de memória (M+, M-, MR, MC).
- Suporte porcentagem relativa ao acumulador atual.
- Adicione um modo somente teclado com indicadores de foco visíveis em cada tecla.

## Definição de Pronto

- [ ] As quatro operações produzem resultados corretos, incluindo sequências encadeadas.
- [ ] O visor nunca mostra `NaN`, `Infinity` ou `undefined`.
- [ ] Limpar reseta totalmente o estado; backspace edita apenas a entrada atual.
- [ ] Teclado e botões da tela se comportam de forma idêntica.
- [ ] Apenas um ponto decimal pode ser inserido por número.

## Armadilhas Comuns

- Usar o texto do visor como fonte da verdade em vez de um modelo de estado separado.
- Concatenar strings para a matemática e obter `"1" + "2" = "12"` em vez de `3`.
- Ignorar o arredondamento de ponto flutuante, fazendo `0.1 + 0.2` mostrar `0.30000000000000004`.
- Esquecer o sinalizador "acabou de avaliar", de modo que o próximo dígito se anexa ao resultado em vez de começar do zero.
- Deixar botões como `<div>`s não focáveis, quebrando o acesso pelo teclado.

## Recursos

- [MDN: Number.prototype.toFixed()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed) — controlar a exibição de decimais.
- [MDN: CSS Grid Layout](https://developer.mozilla.org/pt-BR/docs/Web/CSS/CSS_grid_layout) — dispor o teclado.
- [0.30000000000000004.com](https://0.30000000000000004.com/) — por que a matemática de ponto flutuante se comporta mal.
- [MDN: KeyboardEvent.key](https://developer.mozilla.org/pt-BR/docs/Web/API/KeyboardEvent/key) — mapear teclas físicas para ações.
