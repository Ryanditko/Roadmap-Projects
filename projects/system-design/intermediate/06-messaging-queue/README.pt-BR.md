# Projete uma Fila de Mensagens Distribuída

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete uma fila de mensagens distribuída como Apache Kafka ou RabbitMQ: produtores publicam mensagens em tópicos, consumidores as leem no próprio ritmo, e o sistema retém mensagens de forma durável para que um consumidor lento ou reiniciado não perca nada. Os problemas interessantes são particionar tópicos para paralelismo, rastrear a posição de cada consumidor (offset), replicar para durabilidade e decidir qual garantia de entrega você pode honestamente prometer. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento dos modelos de mensageria pub/sub e baseado em log
- Familiaridade com conceitos de replicação e líder/seguidor
- Noção de semânticas at-most-once, at-least-once e exactly-once
- Conforto para estimar throughput e armazenamento de retenção

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar tópicos, partições e grupos de consumidores para consumo paralelo
- Estimar throughput de escrita, armazenamento de retenção e sobrecarga de replicação
- Projetar o gerenciamento de offsets para consumidores retomarem exatamente de onde pararam
- Raciocinar honestamente sobre garantias de ordenação e entrega
- Justificar trade-offs entre throughput e força da garantia de entrega

## Requisitos e Restrições

- Assuma 1M de mensagens/s de ingestão, média de 1 KB cada, retenção de 7 dias, fator de replicação 3.
- A ordenação é garantida dentro de uma partição, não entre partições.
- Consumidores podem atrasar, reiniciar ou escalar sem perder ou pular mensagens.
- A falha de um broker não pode perder mensagens confirmadas.
- Estime o throughput de escrita em MB/s, o armazenamento total de retenção e o armazenamento replicado.

## Abordagem Sugerida

1. Calcule a banda de ingestão (msgs/s × tamanho) e o armazenamento de retenção (× dias × replicação).
2. Projete o log append-only particionado e como os produtores escolhem uma partição.
3. Projete grupos de consumidores: cada partição consumida por um membro; offsets comitados por grupo.
4. Projete a replicação: líder por partição, seguidores replicam, conjunto de réplicas em sincronia.
5. Escolha uma garantia de entrega e explique o que produtor/consumidor devem fazer para alcançá-la.

## Esboço de Arquitetura

```text
Produtores -> [Cluster de brokers] -> Tópico "orders"
                                        Partição 0 (líder B1, réplicas B2,B3) log append-only
                                        Partição 1 (líder B2, réplicas B1,B3)
                                        ...
Grupo de consumidores "billing":  P0 -> consumidor A   P1 -> consumidor B   (offsets por grupo)

PUB  topic=orders key=userId value=<bytes>   -> ack após replicar para o ISR
SUB  group=billing topic=orders              -> stream a partir do offset comitado
COMMIT group=billing partition=0 offset=12345

Partition { topicId, partId, log[offset -> message], leaderBroker, replicaSet }
Offset    { groupId, topicId, partId, committedOffset }   // particiona por (group, topic, part)
```

## Tópicos de Aprofundamento

- **Particionamento e ordenação:** atribuição de partição por chave; ordenação só dentro de uma partição.
- **Gerenciamento de offset:** offset comitado vs. atual; efeito na reentrega após crash.
- **Trade-off 1 — throughput vs. garantia de entrega:** at-least-once com acks assíncronos é rápido mas reentrega em falhas; exactly-once precisa de produtores idempotentes mais commits transacionais, custando latência. Justifique at-least-once + consumidores idempotentes como padrão pragmático.
- **Trade-off 2 — acks de replicação:** esperar por todas as réplicas maximiza durabilidade mas trava em um seguidor lento; confirmar no conjunto em sincronia equilibra durabilidade e latência.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo a arquitetura acima.
- [ ] Estimativas de capacidade: MB/s de ingestão, armazenamento de retenção, armazenamento replicado, número de partições.
- [ ] Um plano de particionamento ligando chaves a partições e consumidores.
- [ ] Uma estratégia de cache/buffer (page cache, batching) e seu efeito no throughput.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Prometer ordenação global entre partições — é impossível sem uma única partição.
- Comitar offsets antes de processar, transformando at-least-once em perda silenciosa de mensagens.
- Ignorar a sobrecarga de replicação na estimativa de armazenamento (o fator 3 é fácil de esquecer).
- Poucas partições, limitando o paralelismo de consumo por mais consumidores que você adicione.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de mensageria assíncrona.
- [Kafka: The Log (Jay Kreps)](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying-abstraction) — a abstração de log por trás das filas modernas.
- [Documentação do Kafka: design](https://kafka.apache.org/documentation/#design) — particionamento, replicação, semânticas de entrega.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — streams e garantias de entrega.
