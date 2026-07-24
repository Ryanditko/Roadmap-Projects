# Projete um Sistema de Logging Centralizado

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um sistema que coleta logs de muitos serviços, os armazena centralmente e permite que engenheiros os busquem — um ELK stack ou Loki simplificado. A característica definidora são dados pesados de escrita, append-only e de alto volume: logs jorram constantemente, raramente são atualizados e são buscados ocasionalmente. Esse formato guia toda decisão, de como os logs são enviados e bufferizados a como são indexados e envelhecidos. Você vai raciocinar sobre throughput de ingestão, custo de indexação e retenção. Entregue um documento de design cobrindo o pipeline de coleta, o armazenamento e a busca.

## Pré-requisitos

- Entendimento do que é uma linha de log e por que serviços as emitem
- Consciência de que o volume de logs pode ser enorme e em rajadas
- Familiaridade com a ideia de um índice que torna a busca rápida
- Conforto para raciocinar sobre o custo de armazenar dados ao longo do tempo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de coleta que bufferiza rajadas sem perder logs
- Raciocinar sobre o trade-off entre indexar tudo e o custo de armazenamento
- Projetar um layout de armazenamento particionado por tempo
- Planejar retenção e arquivamento para controlar custo
- Enunciar um trade-off entre velocidade de busca e custo de ingestão/armazenamento

## Requisitos e Restrições

1. Coletar logs de muitos serviços e centralizá-los.
2. Absorver rajadas sem descartar logs (buffering/backpressure).
3. Suportar busca por intervalo de tempo, serviço e nível, além de texto livre.
4. Aplicar retenção: logs quentes buscáveis, logs antigos arquivados ou deletados.
5. A ingestão não deve deixar mais lentos os serviços que produzem logs.
6. Estime a escala: 500K linhas de log/s no pico, ~500 bytes cada — cerca de 20 TB/dia.

## Abordagem Sugerida

1. Divida o pipeline: agentes enviam logs, um buffer absorve rajadas, um indexer escreve no armazenamento.
2. Faça a conta: 500K linhas/s × 500 bytes ≈ 250 MB/s ≈ 20 TB/dia; retenção de 7 dias ≈ 140 TB quentes.
3. Projete o buffer (uma fila ou broker de log) para que um indexer lento nunca bloqueie os produtores.
4. Escolha um layout de armazenamento particionado por tempo para que dados antigos sejam baratos de descartar.
5. Decida o que é indexado — texto completo é poderoso mas caro; campos estruturados são mais baratos.

## Esboço de Arquitetura

```text
Serviços ──agente──> [ Buffer / Broker ] ──> [ Indexer ] ──> [ Store Quente (indexado) ]
                        (absorve rajadas)                          │  envelhece
                                                          [ Store Frio / Arquivo ]
Engenheiro ── query ──> [ API de Busca ] ──> Store Quente

API principal
  POST /ingest   (lote de linhas de log)       -> 202
  GET  /search   ?q=&service=&level=&from=&to= -> [ linhas correspondentes ]

Modelo de dados
  linha de log: ts | service | level | message | trace_id | fields{}
  armazenamento particionado por (dia, service); índice em ts, service, level
```

## Tópicos de Aprofundamento

- **Backpressure:** como um buffer/broker desacopla produtores em rajada de um indexer mais estável, e o que acontece quando o buffer enche.
- **Custo de indexação:** índice invertido de texto completo vs. indexar só campos estruturados; o armazenamento e a amplificação de escrita de cada um.
- **Camadas de retenção:** quente (busca rápida) vs. frio/arquivo (barato, lento), e políticas de ciclo de vida para mover dados entre elas.

## Entregáveis

- Um diagrama de arquitetura mostrando agentes, buffer, indexer e armazenamento quente/frio.
- O contrato da API de ingestão e busca.
- Um modelo de dados para uma linha de log e o esquema de particionamento do armazenamento.
- Um trade-off descrito: ex., indexar cada campo (busca rápida e flexível, alto custo de armazenamento e escrita) vs. indexar só campos-chave (barato, mas algumas consultas exigem um scan lento).

## Armadilhas Comuns

- Escrever logs de forma síncrona no store a partir do serviço produtor, acoplando a latência da app ao logging.
- Indexar tudo como texto completo e estourar o orçamento de armazenamento.
- Sem política de retenção, o armazenamento quente cresce sem limite e o custo explode.
- Sem buffer, uma rajada descarta logs ou reflui para os serviços.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de ingestão e armazenamento de dados.
- [Elastic: The ELK Stack](https://www.elastic.co/elastic-stack) — uma arquitetura de logging de referência.
- [Grafana Loki: Arquitetura](https://grafana.com/docs/loki/latest/get-started/architecture/) — um design de logging focado em custo que indexa labels, não texto completo.
- [Twelve-Factor App: Logs](https://12factor.net/logs) — tratar logs como streams de eventos.
