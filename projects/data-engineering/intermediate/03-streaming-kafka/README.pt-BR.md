# Pipeline de Streaming (Kafka)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um pipeline de streaming em tempo real no Apache Kafka: um produtor que emite eventos para um tópico e um consumidor que os lê, mantém estado e escreve resultados em um sink. A mudança de lote para streaming força você a encarar perguntas que o lote nunca faz — o que acontece quando o consumidor cai no meio de uma mensagem, como evitar processar o mesmo evento duas vezes e como computar um agregado sobre um fluxo ilimitado. Você gerenciará os offsets do consumidor deliberadamente, projetará para entrega ao-menos-uma-vez e tornará seu processamento idempotente para que duplicatas sejam inofensivas. Você também adicionará monitoramento de lag do consumidor para enxergar quando o pipeline fica para trás. Este projeto é onde "os dados estão só parados numa tabela" deixa de ser verdade e você começa a pensar em termos de eventos que chegam continuamente.

## Pré-requisitos

- Conforto com uma linguagem que tenha um cliente Kafka (Python, Java ou Go)
- Entendimento de mensageria publish/subscribe e filas
- Familiaridade com JSON ou outro formato de serialização
- Um Kafka local (Docker Compose com Kafka + opcionalmente um schema registry)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Produzir e consumir eventos em um tópico Kafka particionado
- Raciocinar sobre semânticas de entrega: no-máximo-uma-vez, ao-menos-uma-vez, exatamente-uma-vez
- Confirmar offsets deliberadamente para que um crash retome sem perder ou reprocessar dados incorretamente
- Tornar o processamento do consumidor idempotente para que duplicatas ao-menos-uma-vez sejam inofensivas
- Computar um agregado com janela sobre um fluxo ilimitado e monitorar o lag do consumidor

## Requisitos Funcionais

1. Um produtor deve publicar eventos estruturados em um tópico Kafka com uma chave de particionamento.
2. Um consumidor deve ler eventos e escrever resultados derivados em um sink (banco, arquivo ou outro tópico).
3. O consumidor deve confirmar offsets apenas após uma mensagem ser processada com sucesso (ao-menos-uma-vez).
4. O processamento deve ser idempotente para que uma mensagem reentregue não corrompa o sink.
5. Ao reiniciar, o consumidor deve retomar do último offset confirmado — sem lacunas, sem replay completo.
6. O pipeline deve manter um agregado corrente (ex.: contagem ou soma por chave) sobre uma janela de tempo.
7. O lag do consumidor deve ser observável para que ficar para trás seja detectável.

## Marcos Sugeridos

1. **Marco 1 — Produzir e consumir:** Emita eventos para um tópico e os registre a partir de um grupo consumidor.
2. **Marco 2 — Offsets e idempotência:** Confirme após processar e prove que um crash no meio do lote retoma limpo.
3. **Marco 3 — Agregado com janela e lag:** Mantenha um agregado com janela no sink e exponha o lag do consumidor.

## Esboço de Dados e Interface

```text
tópico: page_views   partições: 3   key: user_id

event = {
  "event_id":  "uuid",       # chave de dedup para idempotência
  "user_id":   "u-123",
  "url":       "/pricing",
  "ts":        "2024-01-01T10:00:00Z"
}

Producer --> [ p0 | p1 | p2 ] --> Grupo consumidor "aggregator"
                                     |
                     upsert no sink: (user_id, janela) -> count
                     confirma offset DEPOIS que o upsert tem sucesso

Entrega: ao-menos-uma-vez + upsert idempotente por event_id => efetivamente uma vez
Monitoramento: lag = log-end-offset - offset-confirmado  (por partição)
```

## Desafios Extras

- Adicione uma segunda instância de consumidor e observe o Kafka rebalancear as partições no grupo.
- Roteie mensagens não processáveis para um tópico de dead-letter em vez de bloquear o fluxo.
- Implemente janelas tumbling vs sliding e compare a saída.
- Adicione um schema registry com Avro e evolua o schema do evento sem quebrar consumidores.

## Definição de Pronto

- [ ] Matar o consumidor no meio do fluxo e reiniciá-lo retoma do último offset confirmado.
- [ ] Um evento deliberadamente reentregue não altera o agregado (idempotência verificada).
- [ ] O agregado com janela no sink corresponde a um valor esperado calculado à mão.
- [ ] O lag do consumidor é consultável e sobe e depois se recupera quando o consumidor é pausado.
- [ ] Dois consumidores em um grupo dividem as partições sem processar a mesma mensagem duas vezes.

## Armadilhas Comuns

- Confirmar offsets antes de processar, transformando um crash em perda silenciosa de dados (no-máximo-uma-vez por acidente).
- Auto-commit em um timer enquanto o processamento é lento, fazendo os offsets avançarem além de mensagens não processadas.
- Assumir ordenação entre partições — o Kafka só ordena dentro de uma única partição.
- Escritas não idempotentes no sink, fazendo cada rebalance ou retry inflar o agregado.
- Usar uma única partição "por simplicidade" e depois não conseguir escalar consumidores além de um.

## Recursos

- [Documentação do Apache Kafka](https://kafka.apache.org/documentation/) — brokers, tópicos, partições e grupos consumidores.
- [Kafka: Offsets e semânticas de entrega](https://kafka.apache.org/documentation/#semantics) — ao-menos-uma-vez vs exatamente-uma-vez.
- [Confluent: Consumidores Kafka](https://developer.confluent.io/courses/apache-kafka/consumers/) — gestão de offsets e rebalanceamento explicados.
- [Confluent: Schema Registry e Avro](https://docs.confluent.io/platform/current/schema-registry/index.html) — evoluindo schemas de eventos com segurança.
