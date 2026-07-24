# Agendador Básico (tipo cron)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Pipelines de dados raramente rodam sob demanda — eles rodam *em uma agenda*. Construa um pequeno agendador tipo cron que lê uma lista de jobs, cada um com uma expressão de tempo, e roda cada um quando sua hora chega. Você vai analisar expressões estilo cron, calcular o próximo horário de execução, executar os jobs devidos e manter um histórico do que rodou e se teve sucesso. As partes enganosamente difíceis são as com que todo agendador real se debate: não rodar um job duas vezes no mesmo tick, decidir o que fazer com uma execução perdida, e garantir que um job lento não se sobreponha silenciosamente a si mesmo.

## Pré-requisitos

- Conforto para trabalhar com datas, horas e o conceito de "agora"
- Parsing básico de uma string estruturada (uma expressão cron)
- Capacidade de executar uma função ou comando de shell a partir do código
- I/O de arquivo em nível iniciante para ler definições de jobs e escrever histórico

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Analisar uma expressão estilo cron em uma agenda (minuto, hora, dia, mês, dia da semana)
- Calcular o próximo horário de execução agendado a partir de um dado momento
- Rodar um laço principal que dispara jobs devidos sem busy-waiting
- Garantir que um job rode uma vez por tick agendado, nem zero nem duas vezes
- Registrar o histórico de execução com status, início e duração
- Impedir que um job de longa duração se sobreponha à sua próxima invocação

## Requisitos Funcionais

1. O agendador deve carregar uma lista de jobs, cada um com uma expressão estilo cron e um comando/função a rodar.
2. Deve analisar corretamente a expressão e calcular o próximo horário de execução de cada job.
3. O laço principal deve executar um job exatamente uma vez quando seu horário agendado chega.
4. Nunca deve disparar em dobro um job no mesmo tick, mesmo que o laço esteja levemente atrasado.
5. Se uma execução ainda está em andamento quando a próxima é devida, o agendador deve pulá-la ou enfileirá-la conforme uma política definida (sem sobreposição silenciosa).
6. Deve persistir um histórico de execução: nome do job, horário agendado, início, fim e sucesso/falha.

## Marcos Sugeridos

1. **Marco 1 — Analisar e calcular:** Analise uma expressão cron e imprima os próximos N horários de execução.
2. **Marco 2 — Laço de execução:** Adicione um laço que dispara jobs no horário devido e registra cada execução.
3. **Marco 3 — Correção:** Adicione garantias de uma-vez-por-tick, prevenção de sobreposição e histórico persistido.

## Esboço de Dados e Interface

```text
definições de jobs (arquivo de jobs)
  daily_export   "0 2 * * *"   -> roda export.job
  every_15m      "*/15 * * * *"-> roda sync.job
  weekday_report "30 8 * * 1-5"-> roda report.job

ordem dos campos cron
  minuto hora dia-do-mês mês dia-da-semana

laço do agendador
  now = horário atual (truncado ao minuto)
  para job em jobs:
    se job.next_run <= now e job.last_tick != now:
      roda(job); registra(status, início, duração)
      job.last_tick = now
      job.next_run  = calcula_próximo(job.expr, now)
    se job ainda rodando e devido de novo -> pula (política)
  dorme até o próximo limite de minuto

history.log
  daily_export 2024-05-01T02:00 ok   duração=41s
  sync         2024-05-01T02:15 fail duração=3s "timeout"
```

## Desafios Extras

- Adicionar política de recuperação: na inicialização, decidir se roda jobs que foram perdidos enquanto estava fora do ar.
- Suportar fusos horários para que "0 2 * * *" signifique 2h em um fuso configurado.
- Adicionar retry por job com um número máximo de tentativas em caso de falha.
- Expor um pequeno comando de status listando a última e a próxima execução de cada job.

## Definição de Pronto

- [ ] Expressões cron são analisadas e o cálculo do próximo horário bate com casos verificados à mão.
- [ ] Um job devido roda exatamente uma vez em seu tick agendado.
- [ ] O laço nunca dispara em dobro um job no mesmo minuto.
- [ ] Uma execução sobreposta é pulada ou enfileirada conforme a política documentada.
- [ ] O histórico de execução registra status, tempo e falhas para toda execução.

## Armadilhas Comuns

- Fazer busy-waiting em um laço apertado em vez de dormir até o próximo limite, queimando CPU.
- Disparar em dobro porque o laço verifica o mesmo minuto duas vezes — rastreie o último tick por job.
- Ignorar fusos horários e horário de verão, fazendo as agendas derivarem ou dispararem duas vezes em mudanças de relógio.
- Deixar um job lento se sobrepor à sua próxima execução, fazendo duas cópias mutarem os mesmos dados ao mesmo tempo.

## Recursos

- [crontab.guru](https://crontab.guru/) — decodifique e verifique expressões cron de forma interativa.
- [Wikipedia: Cron](https://en.wikipedia.org/wiki/Cron) — o formato dos campos e sua história.
- [Python `datetime`](https://docs.python.org/3/library/datetime.html) — calcular próximos horários e durações corretamente.
- [Biblioteca croniter](https://github.com/kiorky/croniter) — uma implementação de referência do cálculo de próxima execução para comparar.
