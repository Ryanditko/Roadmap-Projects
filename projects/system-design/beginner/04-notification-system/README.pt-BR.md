# Projete um Sistema de Notificações

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um sistema que entrega notificações por múltiplos canais — email, SMS e push. A ideia central é que a aplicação que produz uma notificação não deve esperar por um terceiro lento e não confiável (um provedor de email ou SMS) responder. Uma fila desacopla os dois: produtores enfileiram, workers desenfileiram e entregam, com retry em caso de falha. Você vai raciocinar sobre confiabilidade, evitar envios duplicados e respeitar preferências do usuário. Entregue um documento de design que explique o fluxo baseado em fila, o modelo de dados e como a entrega sobrevive a falhas de provedores.

## Pré-requisitos

- Entendimento do que uma fila de mensagens faz e por que ela desacopla sistemas
- Consciência de que provedores externos (email/SMS) são lentos e às vezes falham
- Familiaridade com a ideia de um retry e de um processo worker
- Conforto para raciocinar sobre idempotência em alto nível

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de entrega assíncrono e baseado em fila
- Raciocinar sobre retries, backoff e idempotência para evitar envios duplicados
- Modelar múltiplos canais de entrega atrás de uma única interface
- Incorporar preferências do usuário e opt-outs
- Enunciar um trade-off entre velocidade de entrega e confiabilidade

## Requisitos e Restrições

1. Aceitar uma requisição de notificação e entregá-la por um ou mais canais.
2. Suportar ao menos email, SMS e notificações push.
3. Repetir entregas que falharam sem enviar duplicatas ao usuário.
4. Respeitar preferências do usuário (escolha de canal, opt-out, horário de silêncio).
5. A API que aceita requisições deve permanecer rápida mesmo quando um provedor está lento.
6. Estime a escala: 5M notificações/dia entre canais — cerca de 58/s em média com picos.

## Abordagem Sugerida

1. Divida o sistema em uma API de ingestão, uma fila e workers por canal.
2. Faça a conta: 5M/dia ≈ 58/s em média; assuma picos de 5–10× durante campanhas.
3. Projete o caminho de enfileiramento para que aceitar uma requisição nunca bloqueie em um provedor.
4. Projete workers de canal que puxam da fila e chamam o provedor correto.
5. Adicione uma chave de idempotência e rastreamento de status de entrega para que retries não enviem em dobro.

## Esboço de Arquitetura

```text
Produtor ── POST /notify ──> [ API de Ingestão ] ──> [ Fila ]
                                                        │
                    ┌────────────────────────────────────┼──────────────────┐
              [ Worker Email ]                    [ Worker SMS ]      [ Worker Push ]
                    │                                   │                    │
              provedor email                      provedor SMS         provedor push
                    └──────────> [ Log de Entrega ] <────────────────────────┘

API principal
  POST /notify  { userId, template, channels, data, idempotencyKey }
                -> 202 { notificationId }
  GET  /notify/{id}                       -> { status por canal }

Modelo de dados
  notifications: id (PK) | user_id | template | payload | idempotency_key | created_at
  deliveries:    notification_id | channel | status | attempts | last_attempt_at
  preferences:   user_id | channel | enabled | quiet_hours
```

## Tópicos de Aprofundamento

- **Idempotência:** como uma chave de idempotência mais uma verificação de status previne envios duplicados no retry.
- **Estratégia de retry:** backoff exponencial, máximo de tentativas e roteamento de mensagens esgotadas para uma dead-letter queue.
- **Priorização:** filas separadas para notificações transacionais (OTP) vs. em massa (marketing).

## Entregáveis

- Um diagrama de arquitetura mostrando ingestão, fila e workers por canal.
- O contrato da API principal para submeter e checar uma notificação.
- Um modelo de dados para notificações, entregas e preferências.
- Um trade-off descrito: ex., envio síncrono (simples, feedback imediato, frágil sob latência do provedor) vs. assíncrono baseado em fila (resiliente, escalável, mas eventual e com mais peças móveis).

## Armadilhas Comuns

- Chamar o provedor de email/SMS de forma síncrona no caminho da requisição, fazendo a latência do provedor virar a sua.
- Repetir sem idempotência, de modo que uma falha transitória faz o usuário receber três cópias.
- Ignorar preferências e horários de silêncio do usuário, gerando spam e reclamações de opt-out.
- Não ter um caminho de dead-letter, de modo que mensagens permanentemente falhas repetem para sempre.

## Recursos

- [System Design Primer: Assincronismo](https://github.com/donnemartin/system-design-primer#asynchronism) — filas e workers explicados.
- [AWS SQS: Dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) — tratando mensagens que não podem ser entregues.
- [Stripe: APIs robustas e previsíveis com idempotência](https://stripe.com/blog/idempotency) — chaves de idempotência na prática.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — padrões de backoff e load-shedding.
