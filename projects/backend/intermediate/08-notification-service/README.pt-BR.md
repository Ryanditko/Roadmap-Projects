# Serviço de Notificações (email + retentativa)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa o serviço que todo produto acaba precisando: um lugar central que outros sistemas chamam para dizer "notifique este usuário", sem se importar como. Por trás de uma única API você se ramifica para múltiplos canais — email, SMS, push — cada um apoiado por um provedor que pode falhar, aplicar rate-limit ou ficar em silêncio. Seu trabalho é aceitar a requisição rapidamente, entregar de forma assíncrona, retentar as falhas transitórias com backoff, rastrear o que de fato chegou e respeitar as preferências e os horários de silêncio do usuário. A engenharia interessante não é enviar um email; é garantir que a entrega seja tentada de forma confiável sem nunca notificar um usuário duas vezes pelo mesmo evento.

## Pré-requisitos

- Conforto para construir endpoints REST e workers em segundo plano
- Familiaridade com uma fila de mensagens ou ao menos uma tabela de jobs durável ([Sistema de Fila de Jobs (RabbitMQ)](../04-job-queue-rabbitmq/) é um forte complemento)
- Entendimento de processamento assíncrono e por que se desacopla aceitar de entregar
- Acesso a um provedor de email em sandbox (ou um coletor SMTP local como o MailHog) para testes

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma API agnóstica de canal que esconde os detalhes do provedor atrás de uma interface comum
- Implementar um pipeline de envio durável com backoff exponencial e um caminho de dead-letter
- Rastrear o status de entrega de cada notificação por todo o seu ciclo de vida
- Renderizar templates com substituição de variáveis e conteúdo por localidade
- Honrar preferências do usuário, opt-outs e janelas de não-perturbe antes de enviar

## Requisitos Funcionais

1. O sistema deve aceitar uma requisição de notificação nomeando um destinatário, um evento/template e variáveis de payload, e retornar imediatamente com um ID de rastreamento.
2. A entrega deve ocorrer de forma assíncrona via workers, nunca inline com a requisição de aceite.
3. O sistema deve suportar ao menos dois canais por trás de uma interface de envio comum, escolhida por requisição ou por preferência do usuário.
4. Um envio falho deve ser retentado com backoff exponencial até um limite de tentativas, e então movido para um armazenamento de dead-letter.
5. O sistema deve persistir e expor o status de entrega de cada notificação (na fila, enviado, entregue, falhou).
6. Templates devem suportar interpolação de variáveis e ser validados para que uma variável ausente falhe cedo, não no meio do envio.
7. O sistema deve verificar preferências do usuário e períodos de não-perturbe, suprimindo ou adiando envios que os violem.
8. O sistema deve ser idempotente por (destinatário, chave de evento) para que uma requisição duplicada não notifique duas vezes.

## Marcos Sugeridos

1. **Marco 1 — Aceitar e entregar:** Aceite requisições, enfileire-as e tenha um worker entregando email por um provedor com rastreamento de status.
2. **Marco 2 — Retentativa e canais:** Adicione backoff exponencial, um armazenamento de dead-letter e um segundo canal por trás da interface compartilhada.
3. **Marco 3 — Templates e preferências:** Adicione renderização de template, preferências do usuário, não-perturbe e idempotência por evento.

## Esboço de Dados e Interface

```text
Notification
  id:            string
  recipientId:   string
  channel:       enum { email, sms, push }
  templateId:    string
  status:        enum { queued, sending, sent, delivered, failed, suppressed }
  attempts:      inteiro
  eventKey:      string (para idempotência)
  createdAt:     string ISO-8601

Preferences
  userId:        string
  channels:      { email: true, sms: false, push: true }
  quietHours:    { start: "22:00", end: "07:00", tz: "America/Sao_Paulo" }

POST /notifications   body: { recipientId, templateId, channel?, vars, eventKey }
                      -> 202 { id, status: "queued" }
GET  /notifications/{id}  -> 200 { status, attempts, ... } | 404

Agenda de retentativa (backoff): tentativa n -> atraso = base * 2^(n-1) (+ jitter)
Após máximo de tentativas -> armazenamento de dead-letter
```

## Desafios Extras

- Adicione canais de fallback: se o push falhar permanentemente, tente email automaticamente.
- Suporte envios em lote que expandem uma requisição em muitos destinatários de forma eficiente.
- Adicione rate limiting por canal para nunca exceder o teto de vazão de um provedor.
- Gere análises de entrega: taxas de enviado/entregue/falhou por canal ao longo do tempo.

## Definição de Pronto

- [ ] O endpoint de aceite retorna imediatamente; a entrega sempre acontece em um worker.
- [ ] Falhas transitórias retentam com backoff e jitter, e vão para dead-letter após o limite.
- [ ] O status de entrega é consultável e reflete o ciclo de vida real de cada notificação.
- [ ] Templates rejeitam variáveis ausentes antes de um envio ser tentado.
- [ ] Preferências e horários de silêncio suprimem ou adiam envios corretamente, e duplicatas são colapsadas por chave de evento.

## Armadilhas Comuns

- Enviar dentro da requisição HTTP, de modo que um provedor lento deixa sua API lenta e derruba notificações no crash.
- Retentar sem jitter, causando um thundering herd que martela um provedor em recuperação.
- Retentar falhas não-transitórias (um hard bounce, um número inválido) para sempre em vez de mandá-las para dead-letter.
- Tratar "provedor aceitou" como "entregue" — aceito só significa enfileirado no provedor; use os webhooks de entrega dele para o status real.
- Ignorar idempotência, de modo que uma retentativa do cliente notifica o usuário em dobro.

## Recursos

- [Twilio: Status de entrega e callbacks](https://www.twilio.com/docs/messaging/guides/track-outbound-message-status) — como o rastreamento de entrega real funciona.
- [SendGrid: Event webhook](https://www.twilio.com/docs/sendgrid/for-developers/tracking-events/event) — enviado vs entregue vs bounced.
- [AWS: Retentativas de erro e backoff exponencial](https://docs.aws.amazon.com/general/latest/gr/api-retries.html) — backoff e jitter feitos direito.
- [MailHog](https://github.com/mailhog/MailHog) — um servidor SMTP local para testar email sem enviar mensagens reais.
- [RabbitMQ: Dead Letter Exchanges](https://www.rabbitmq.com/docs/dlx) — o padrão de dead-letter clássico.
