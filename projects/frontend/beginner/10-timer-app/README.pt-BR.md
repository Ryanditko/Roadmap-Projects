# Aplicativo de Temporizador

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um app de temporizador com dois modos: um cronômetro que conta para cima e uma contagem regressiva que conta até zero e alerta. Parece simples, mas ensina a coisa mais mal compreendida sobre tempo no navegador — você não pode confiar que o `setInterval` seja preciso. Intervalos derivam, e eles desaceleram ou pausam totalmente em abas em segundo plano. A abordagem correta é armazenar um timestamp de início e calcular o tempo decorrido a partir do relógio real a cada tique, usando o intervalo apenas para disparar repinturas. Acerte isso e seu temporizador permanece preciso mesmo depois de a aba ter ficado oculta por um minuto.

## Pré-requisitos

- Fundamentos de HTML, CSS e JavaScript
- `setInterval`, `clearInterval` e `Date.now()`
- Modelagem básica de estado (rodando vs. pausado vs. parado)
- Formatar números em strings `MM:SS`

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Rastrear o tempo decorrido a partir de um timestamp real em vez de acumular tiques de intervalo
- Implementar iniciar, pausar, retomar e resetar sem deriva
- Construir os modos cronômetro (contar para cima) e contagem regressiva (contar para baixo)
- Formatar o tempo de forma consistente e atualizar o visor suavemente
- Disparar um alerta/notificação acessível quando uma contagem regressiva chega a zero

## Requisitos Funcionais

1. O cronômetro conta para cima a partir do zero e pode ser pausado, retomado e resetado.
2. A contagem regressiva aceita uma duração, conta até zero e sinaliza a conclusão.
3. O tempo decorrido/restante é calculado a partir de timestamps reais, permanecendo preciso após uma aba em segundo plano.
4. O tempo é exibido em um formato claro `MM:SS` (ou `HH:MM:SS`).
5. Pausar preserva o tempo decorrido exato; retomar continua a partir dele.
6. Resetar retorna ao estado inicial com os controles nos estados corretos de habilitado/desabilitado.
7. A conclusão da contagem regressiva é anunciada de forma acessível, não apenas pela cor.

## Marcos Sugeridos

1. **Marco 1 — Cronômetro:** Conte para cima a partir de um timestamp de início com iniciar/pausar/resetar.
2. **Marco 2 — Contagem regressiva:** Aceite uma duração e conte para baixo, sinalizando o zero.
3. **Marco 3 — Precisão e polimento:** Verifique a ausência de deriva ao colocar a aba em segundo plano; adicione o alerta de conclusão.

## Esboço de Dados e Interface

```text
Estado do timer
  mode:       "stopwatch" | "countdown"
  status:     "idle" | "running" | "paused"
  startedAt:  number | null   (Date.now() no último início)
  baseElapsed: number         (ms acumulados antes da execução atual)
  durationMs: number          (alvo da contagem regressiva)

elapsed = baseElapsed + (running ? Date.now() - startedAt : 0)

Layout
+------------------------------------------+
|            00:00:00                      |  <- visor
+------------------------------------------+
|  [ Iniciar ]  [ Pausar ]  [ Resetar ]    |
+------------------------------------------+
|  Modo: (o) Cronômetro  ( ) Regressiva    |
+------------------------------------------+
```

## Desafios Extras

- Adicione cronometragem de voltas ao cronômetro, registrando tempos parciais em uma lista.
- Adicione presets Pomodoro que alternam contagens de trabalho e pausa.
- Use a Notification API para alertar quando a aba não está focada.
- Adicione um anel de progresso circular que preenche conforme a contagem regressiva corre (respeite movimento reduzido).

## Definição de Pronto

- [ ] O tempo decorrido é derivado de timestamps e permanece preciso após o segundo plano.
- [ ] Iniciar, pausar, retomar e resetar se comportam corretamente com estados de controle correspondentes.
- [ ] Ambos os modos cronômetro e contagem regressiva funcionam de forma independente.
- [ ] A conclusão da contagem regressiva é anunciada sem depender apenas da cor.
- [ ] O tempo é sempre formatado de forma consistente, sem valores negativos ou `NaN`.

## Armadilhas Comuns

- Incrementar um contador a cada tique de `setInterval`, que deriva e estagna em abas em segundo plano.
- Esquecer o `clearInterval`, vazando temporizadores e empilhando atualizações.
- Não preservar o tempo decorrido entre pausar/retomar, resetando o relógio acidentalmente.
- Sinalizar a conclusão apenas com uma mudança de cor, invisível para alguns usuários.
- Exibir tempo negativo quando a contagem regressiva ultrapassa o zero em um tique lento.

## Recursos

- [MDN: setInterval()](https://developer.mozilla.org/pt-BR/docs/Web/API/setInterval) — suas garantias e seus limites.
- [MDN: Date.now()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Date/now) — o timestamp do qual sua precisão depende.
- [MDN: Page Visibility API](https://developer.mozilla.org/pt-BR/docs/Web/API/Page_Visibility_API) — saber quando uma aba está oculta.
- [MDN: ARIA live regions](https://developer.mozilla.org/pt-BR/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — anunciar a conclusão de forma acessível.
