# Sistema de Fila de Jobs (RabbitMQ)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a maquinaria que permite a uma API dizer "faça isso depois" e seguir com a vida. Um cliente submete um job — redimensionar uma imagem, enviar um relatório, cobrar um cartão — e seu serviço o aceita instantaneamente, entrega a um message broker e retorna um ID de rastreamento. Processos worker separados puxam jobs da fila e os executam, no próprio ritmo, nas próprias máquinas. O trabalho interessante é tudo o que cerca o caminho feliz: retentar um job que falhou por um motivo transitório, desistir graciosamente de um que nunca vai funcionar, rastrear o status para que um chamador possa consultar, e garantir que um crash de worker nunca perca um job silenciosamente. O RabbitMQ te dá as primitivas — acknowledgements, dead-letter exchanges, prefetch — e o seu design decide se o sistema é confiável ou furado.

## Pré-requisitos

- Conforto para construir uma API HTTP e rodar um processo de longa duração em segundo plano
- Entendimento de por que se desacopla aceitar o trabalho de fazer o trabalho
- Uma instância do RabbitMQ rodando (Docker é o caminho mais fácil) e uma biblioteca cliente para sua linguagem
- Familiaridade com JSON ou outro formato de serialização para os payloads das mensagens

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um pipeline produtor/consumidor sobre um message broker durável
- Usar acknowledgements e prefetch para que um worker que caiu nunca perca o trabalho em andamento
- Implementar retentativas limitadas com backoff exponencial e um caminho de dead-letter
- Projetar jobs idempotentes para que uma mensagem reentregue não cause dano
- Rastrear e expor o status de um job por todo o seu ciclo de vida
- Desligar um worker graciosamente sem descartar o job que ele está processando

## Requisitos Funcionais

1. A API deve aceitar um job com um tipo e payload, enfileirá-lo de forma durável e retornar um ID de rastreamento imediatamente.
2. Workers devem consumir jobs de forma assíncrona e confirmar (ack) apenas após o processamento bem-sucedido.
3. Um job que falha transitoriamente deve ser retentado com backoff exponencial até um limite de tentativas.
4. Um job que esgota suas retentativas deve cair em uma dead-letter queue, nunca ser descartado silenciosamente.
5. O sistema deve persistir e expor o status de cada job (na fila, processando, sucesso, falhou, dead-lettered).
6. O processamento de jobs deve ser idempotente, para que uma mensagem reentregue não execute efeitos colaterais em dobro.
7. Workers devem suportar desligamento gracioso, finalizando ou reenfileirando jobs em andamento antes de sair.
8. A fila deve sobreviver a um restart do broker (filas duráveis e mensagens persistentes).

## Marcos Sugeridos

1. **Marco 1 — Submeter e processar:** Aceite jobs via HTTP, publique em uma fila durável e tenha um worker consumindo e dando ack neles.
2. **Marco 2 — Retentativa e dead-letter:** Adicione retentativas limitadas com backoff e roteie jobs esgotados para uma dead-letter queue.
3. **Marco 3 — Status e ciclo de vida:** Persista o status do job, exponha um endpoint de status e adicione idempotência mais desligamento gracioso.

## Esboço de Dados e Interface

```text
Job
  id:         string
  type:       string   (ex.: "resize-image")
  payload:    object
  status:     enum { queued, processing, succeeded, failed, dead }
  attempts:   inteiro
  maxRetries: inteiro
  createdAt:  string ISO-8601

POST /jobs          body: { type, payload }
                    -> 202 { id, status: "queued" }
GET  /jobs/{id}     -> 200 { status, attempts, ... } | 404

Topologia do broker
  jobs.exchange -> jobs.queue        (workers consomem, ack manual)
  on nack/expire -> jobs.dlx -> jobs.deadletter.queue

Retentativa: republicar com atraso = base * 2^(tentativa-1) (+ jitter), até maxRetries
```

## Desafios Extras

- Adicione níveis de prioridade para que jobs urgentes furem a fila de jobs de baixa prioridade.
- Suporte jobs atrasados/agendados que só ficam disponíveis em um momento futuro.
- Adicione heartbeats de worker e um dashboard mostrando a profundidade da fila e a saúde dos workers.
- Implemente dependências entre jobs, onde um job só roda depois que outro tem sucesso.

## Definição de Pronto

- [ ] Submeter um job retorna imediatamente com um ID de rastreamento; o trabalho acontece em um worker.
- [ ] Um crash de worker no meio de um job faz a mensagem ser reentregue, não perdida (verificado matando um worker).
- [ ] Falhas transitórias retentam com backoff; jobs esgotados ficam visíveis na dead-letter queue.
- [ ] O status do job é consultável e reflete o ciclo de vida real.
- [ ] Reentregar o mesmo job não duplica seus efeitos colaterais.

## Armadilhas Comuns

- Dar auto-ack nas mensagens no recebimento, de modo que um crash entre receber e finalizar perde o job.
- Filas não-duráveis ou mensagens não-persistentes, de modo que um restart do broker apaga o backlog.
- Retentar falhas não-transitórias para sempre em vez de mandá-las para dead-letter após um limite.
- Prefetch ilimitado, de modo que um worker agarra a fila inteira e mata de fome os outros.
- Assumir entrega exatamente-uma-vez — o RabbitMQ é pelo-menos-uma-vez, então jobs devem ser idempotentes.

## Recursos

- [RabbitMQ: Tutoriais](https://www.rabbitmq.com/tutorials) — work queues, acknowledgements e roteamento na fonte.
- [RabbitMQ: Dead Letter Exchanges](https://www.rabbitmq.com/docs/dlx) — o padrão de dead-letter clássico.
- [RabbitMQ: Consumer Prefetch](https://www.rabbitmq.com/docs/consumer-prefetch) — dispatch justo e como evitar starvation de workers.
- [AWS: Retentativas de erro e backoff exponencial](https://docs.aws.amazon.com/general/latest/gr/api-retries.html) — backoff e jitter feitos direito.
- [Enterprise Integration Patterns: Dead Letter Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html) — o conceito por trás da fila.
