# Arquitetura de Microsserviços Orientada a Eventos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Em um sistema orientado a requisições, os serviços chamam uns aos outros diretamente e cada dependência é um potencial ponto de acoplamento e falha. Em um sistema **orientado a eventos**, os serviços anunciam fatos — "PedidoRealizado", "PagamentoRecebido", "UsuárioRegistrado" — para um log compartilhado, e serviços interessados reagem no seu próprio ritmo. O publicador não sabe nem se importa com quem consome. Esse desacoplamento te dá deploy independente, buffering natural sob carga e a capacidade de adicionar novos consumidores sem tocar nos produtores. Também te entrega um novo conjunto de problemas: eventos chegam fora de ordem, esquemas evoluem, consumidores caem no meio de um lote, e "o que de fato aconteceu?" vira uma pergunta que você responde reprocessando um log. Este projeto pede que você construa um sistema orientado a eventos pequeno mas honesto e enfrente cada uma dessas realidades.

## Pré-requisitos

- Experiência construindo serviços HTTP independentes ([Fila de Tarefas com RabbitMQ](../../intermediate/04-job-queue-rabbitmq/) e [Serviço de Notificações](../../intermediate/08-notification-service/) são boas preparações)
- Um message broker ou log de eventos que você possa rodar localmente (Kafka, Redpanda, RabbitMQ ou NATS)
- Conforto com formatos de serialização (JSON e, idealmente, Avro ou Protobuf)
- Entendimento básico das semânticas de entrega at-least-once vs. exactly-once
- Qualquer stack de backend; duas linguagens diferentes entre serviços é um bônus para provar o desacoplamento

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar eventos como fatos imutáveis e autodescritivos e separar leituras de escritas (CQRS)
- Versionar um esquema de evento e evoluí-lo sem quebrar consumidores existentes
- Construir consumidores idempotentes e tolerantes a duplicatas e reordenação
- Reprocessar eventos para reconstruir estado ou integrar um novo serviço
- Conter falhas com filas de mensagens mortas (DLQ) e observabilidade por serviço

## Requisitos Funcionais

1. Pelo menos três serviços independentes devem se comunicar exclusivamente por eventos publicados — sem chamadas síncronas diretas no fluxo principal.
2. Eventos devem ser imutáveis, carregar uma versão de esquema e ser armazenados em um log append-only ou broker.
3. Uma mudança de esquema deve ser introduzida sem quebrar consumidores já implantados (evolução aditiva/retrocompatível).
4. Consumidores devem ser idempotentes: reprocessar um evento não deve corromper estado nem duplicar efeitos colaterais.
5. O sistema deve suportar replay — um consumidor novo ou resetado pode reconstruir seu estado a partir do início do log.
6. Mensagens venenosas (poison) devem ser roteadas para uma dead-letter queue com contexto suficiente para diagnosticá-las.
7. **Consistência:** o sistema é eventualmente consistente; documente as janelas de read-your-writes e onde o dado desatualizado fica visível.
8. **Disponibilidade:** um consumidor fora do ar não deve bloquear produtores; eventos ficam em buffer e são processados na recuperação.
9. **Observabilidade:** cada serviço deve expor seu lag de consumo e métricas de processamento, e eventos devem carregar um correlation ID.

## Marcos Sugeridos

1. **Marco 1 — Publicar e assinar:** Um produtor, um broker, dois consumidores reagindo a um stream de eventos compartilhado.
2. **Marco 2 — Esquema e versionamento:** Adicione um schema registry ou campo de versão; evolua um evento e prove que consumidores antigos ainda funcionam.
3. **Marco 3 — Resiliência:** Adicione idempotência, tratamento de dead-letter e offsets de grupo de consumo; recupere um consumidor que caiu.
4. **Marco 4 — Replay e CQRS:** Reconstrua um modelo de leitura a partir do log e separe o caminho de escrita do de consulta.

## Esboço de Dados e Interface

```text
  ┌──────────┐  PedidoRealizado ┌───────────────────────────┐
  │ Produtor │ ───────────────▶ │   Log de Eventos / Broker  │
  └──────────┘                  │   (append-only, versionado)│
                                └───────┬─────────┬──────────┘
                              consome   │         │  consome
                                ┌───────▼──┐  ┌───▼────────┐
                                │ Cobrança │  │ Modelo de  │──▶ API de consulta
                                │          │  │  Leitura   │
                                └──────────┘  └────────────┘
                                     │ poison
                                ┌────▼─────┐
                                │  DLQ     │
                                └──────────┘

Envelope de evento
  eventId, type, schemaVersion, occurredAt, correlationId, payload{...}
```

## Desafios Extras

- Introduza um schema registry (ex.: Confluent/Apicurio) e imponha compatibilidade na publicação.
- Adicione o padrão outbox para que a escrita no banco de um serviço e a publicação do evento não possam divergir.
- Implemente event sourcing para um agregado e derive múltiplas projeções do mesmo stream.
- Adicione uma saga/process manager coordenando um fluxo multisserviço puramente por eventos.

## Definição de Pronto

- [ ] O fluxo principal roda com zero chamadas HTTP diretas serviço-a-serviço.
- [ ] Um esquema de evento evoluído é consumido tanto pela versão antiga quanto pela nova do consumidor sem erro.
- [ ] Reprocessar o log reconstrói o estado de um consumidor identicamente ao seu estado processado ao vivo.
- [ ] Um consumidor que caiu se atualiza a partir do seu último offset após reiniciar, sem eventos perdidos.
- [ ] Mensagens venenosas aparecem na DLQ com correlation IDs, não descartadas silenciosamente.

## Armadilhas Comuns

- Enfiar uma chamada síncrona entre serviços "só desta vez" — isso reintroduz o acoplamento que os eventos deveriam remover.
- Fazer eventos que são comandos disfarçados ("FaçaIssoAgora") em vez de fatos ("IssoAconteceu").
- Quebrar consumidores com uma mudança de esquema não aditiva (renomear ou remover um campo) em vez de evoluir de forma compatível.
- Assumir entrega exactly-once; a maioria dos brokers dá at-least-once, então idempotência é obrigatória, não opcional.

## Recursos

- [Martin Fowler: What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html) — desembaraça os quatro significados distintos do termo.
- [Confluent: Event sourcing, CQRS e stream processing](https://www.confluent.io/learn/event-sourcing/) — padrões e seus trade-offs.
- [Confluent: Evolução e compatibilidade de esquemas](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html) — como mudar esquemas com segurança.
- [Microservices.io: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html) — publicando eventos de forma confiável junto com escritas no banco.
- [Documentação do Apache Kafka](https://kafka.apache.org/documentation/) — logs, partições, grupos de consumo e offsets.
