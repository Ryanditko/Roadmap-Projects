# Projete um Agendador de Tarefas

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um sistema que executa tarefas — algumas agendadas (toda noite às 2h), algumas uma única vez no futuro, algumas como jobs de background enfileirados por outros serviços. Os problemas interessantes são garantir que uma tarefa rode mesmo se um worker travar no meio da execução, e garantir que ela rode exatamente uma vez em vez de duas quando workers competem. Você vai raciocinar sobre um store durável de tarefas, coordenação de workers, retries e visibility timeouts. Entregue um documento de design cobrindo o modelo de agendamento, o fluxo dos workers e as garantias de confiabilidade.

## Pré-requisitos

- Entendimento da diferença entre um job agendado e um job sob demanda
- Consciência de que um worker pode travar segurando uma tarefa
- Familiaridade com filas e a ideia de "reivindicar" trabalho
- Conforto para raciocinar sobre execução at-least-once vs. exactly-once

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um store durável de tarefas que sobrevive a travamentos de workers
- Raciocinar sobre como workers reivindicam tarefas sem dois rodarem a mesma
- Tratar retries e tarefas falhas com backoff e um caminho de dead-letter
- Suportar tarefas recorrentes estilo cron e tarefas atrasadas de disparo único
- Enunciar um trade-off entre execução exactly-once e at-least-once

## Requisitos e Restrições

1. Agendar uma tarefa para rodar uma vez no futuro ou em um cron recorrente.
2. Uma tarefa deve rodar mesmo se o worker que a pegou travar.
3. Evitar rodar a mesma tarefa duas vezes quando múltiplos workers competem.
4. Repetir tarefas falhas com backoff; desviar tarefas permanentemente falhas.
5. Expor o status da tarefa (pendente, rodando, sucesso, falha).
6. Estime a escala: 1M tarefas/dia, até 200 workers concorrentes.

## Abordagem Sugerida

1. Separe o scheduler (decide o que está devido) dos workers (executam tarefas).
2. Faça a conta: 1M/dia ≈ 12 tarefas/s em média; projete para picos quando muitos jobs cron disparam no mesmo minuto (ex., meia-noite).
3. Projete a reivindicação de tarefa: um worker marca atomicamente uma tarefa como "rodando" com um visibility timeout para que um travamento a libere de volta.
4. Projete retries com backoff e um teto de máximo de tentativas em um store de dead-letter.
5. Explique como um índice por horário de execução permite ao scheduler achar tarefas prontas de forma barata.

## Esboço de Arquitetura

```text
[ Scheduler ] --varre tarefas devidas--> [ Store de Tarefas ] <--claim/ack-- [ Pool de Workers ]
   (cron + disparo único)                       │                              (N workers)
                                          atualizações de status
                                               │
                          esgotadas -> [ Store de Dead-letter ]

API principal
  POST /tasks   { runAt | cron, type, payload }  -> 201 { taskId }
  GET  /tasks/{id}                                -> { status, attempts, lastError }
  DELETE /tasks/{id}                              -> 204  (cancelar)

Modelo de dados
  tasks: task_id (PK) | type | payload | run_at | cron | status
         | attempts | locked_by | lock_expires_at
  índice em (status, run_at) para achar tarefas devidas e não reivindicadas
```

## Tópicos de Aprofundamento

- **Visibility timeout:** como um lease com expiração permite que a tarefa de um worker travado seja re-reivindicada automaticamente.
- **Exactly-once vs. at-least-once:** por que exactly-once de verdade é difícil, e como tarefas idempotentes tornam at-least-once seguro.
- **Thundering herd à meia-noite:** espalhar tarefas recorrentes com jitter para que não disparem todas simultaneamente.

## Entregáveis

- Um diagrama de arquitetura mostrando scheduler, store de tarefas, pool de workers e caminho de dead-letter.
- O contrato da API principal para agendar, checar e cancelar tarefas.
- Um modelo de dados para tarefas incluindo campos de locking.
- Um trade-off descrito: ex., at-least-once com tarefas idempotentes (simples, robusto, pode executar em dobro) vs. tentar exactly-once (sem duplicatas, mas coordenação complexa e ainda não perfeita).

## Armadilhas Comuns

- Assumir que uma tarefa roda até o fim — projetar sem um caminho de recuperação de travamento.
- Deixar dois workers reivindicarem a mesma tarefa porque a reivindicação não é atômica.
- Repetir tarefas não-idempotentes, de modo que um retry causa um efeito colateral duplicado (ex., cobrança dupla).
- Todos os jobs cron disparando exatamente à meia-noite e sobrecarregando os workers de uma vez.

## Recursos

- [System Design Primer: Assincronismo](https://github.com/donnemartin/system-design-primer#asynchronism) — filas de tarefas e workers.
- [AWS SQS: Visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) — como leases permitem recuperação de travamento.
- [Crontab guru](https://crontab.guru/) — referência de sintaxe de agendamento cron.
- [Google SRE Book: Distributed cron](https://sre.google/sre-book/chapters/) — rodar jobs agendados de forma confiável em escala.
