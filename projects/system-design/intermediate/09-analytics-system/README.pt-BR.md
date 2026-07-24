# Projete um Sistema de Analytics

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de um sistema de analytics de produto como Mixpanel ou Google Analytics: apps clientes emitem uma torrente de eventos comportamentais, o sistema os ingere de forma confiável, e analistas consultam agregações ("usuários ativos diários", "conversão de funil") sobre bilhões de linhas sem esperar minutos. O design se divide em um caminho de ingestão de alto throughput e um store analítico otimizado para consulta, normalmente com uma camada em tempo real e uma em batch. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento de fluxos de eventos e cargas OLTP vs. OLAP
- Familiaridade com armazenamento colunar e pré-agregação em nível conceitual
- Noção de filas de mensagens e processamento de streams
- Conforto para estimar volume de eventos, throughput de ingestão e armazenamento

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de ingestão que bufferiza e agrupa um fluxo de eventos de alto volume
- Estimar QPS de escrita de eventos, armazenamento bruto e armazenamento colunar ao longo do tempo
- Escolher um store colunar/analítico e projetar pré-agregações para consultas comuns
- Projetar uma divisão estilo lambda entre camadas em tempo real e batch
- Justificar trade-offs entre armazenamento de eventos brutos e rollups pré-agregados

## Requisitos e Restrições

- Assuma 1M de eventos/s no pico, média de 500 bytes cada, retenção bruta de 90 dias, dashboards atualizados quase em tempo real.
- A ingestão não pode perder eventos se o store analítico ficar brevemente indisponível.
- Consultas comuns de dashboard devem retornar em menos de ~1 s sobre bilhões de eventos.
- Suporte tanto dashboards predefinidos quanto consultas ad-hoc mais lentas.
- Estime o throughput de ingestão (MB/s), o armazenamento bruto e o armazenamento de rollups.

## Abordagem Sugerida

1. Calcule a banda de ingestão de eventos (eventos/s × tamanho) e o armazenamento bruto de 90 dias.
2. Projete a ingestão: coletor → fila durável → processador de stream → store colunar.
3. Projete a pré-agregação: consolide eventos em cubos horários/diários para dashboards.
4. Separe a camada em tempo real (aproximada, recente) da batch (exata, histórica).
5. Particione o store analítico por tempo e por uma dimensão de alta cardinalidade.

## Esboço de Arquitetura

```text
Clientes -> [API Coletor] -> Kafka (buffer durável, particionado por eventType)
                                |-> Processador de stream -> rollups em tempo real (últimas horas)
                                |-> Carregador batch      -> Store colunar (tipo ClickHouse/BigQuery)
                                                               |-> tabelas de rollup diárias (cubos)

Consulta de dashboard -> [svc Consulta] -> tabelas de rollup (rápido) OU eventos brutos (ad-hoc, lento)

POST /collect   { userId, event, props{}, ts } -> 202 (aceito, assíncrono)
GET  /metrics?metric=DAU&from&to&groupBy       -> 200 { series[] }

Event  { eventId, userId, type, props{}, ts }   // particiona por (dia, type)
Rollup { day, dimension, metric -> value }      // cubo pré-agregado
```

## Tópicos de Aprofundamento

- **Durabilidade da ingestão:** bufferizar em uma fila para que uma queda a jusante nunca descarte eventos.
- **Pré-agregação:** cubos de rollup vs. consultar eventos brutos; explosão de cardinalidade de group-bys.
- **Trade-off 1 — eventos brutos vs. rollups pré-agregados:** manter eventos brutos permite qualquer consulta futura mas é caro de armazenar e lento de escanear; rollups são minúsculos e rápidos mas só respondem perguntas predefinidas. Justifique manter brutos por 90 dias mais rollups para dashboards.
- **Trade-off 2 — precisão em tempo real vs. batch:** a camada de streaming dá números frescos mas aproximados; a camada batch é exata mas atrasada. Justifique servir janelas recentes do streaming e o histórico do batch.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo o pipeline e a arquitetura acima.
- [ ] Estimativas de capacidade: MB/s de ingestão, armazenamento bruto (90 dias), armazenamento de rollups.
- [ ] Um plano de particionamento para os stores de eventos e rollups (por tempo + dimensão).
- [ ] Uma estratégia de cache/pré-agregação para consultas de dashboard.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Escrever eventos direto no store analítico, fazendo qualquer soluço a jusante perder dados.
- Armazenar apenas rollups e depois não conseguir responder uma nova pergunta sobre o comportamento passado.
- Ignorar a cardinalidade dos group-by, fazendo um rollup por user-agent explodir para milhões de linhas.
- Usar um store por linha para analytics, tornando varreduras completas irremediavelmente lentas.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — pipelines e trade-offs de armazenamento.
- [Lambda Architecture (Marz)](http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html) — o padrão de camada batch + velocidade.
- [ClickHouse: por que colunar](https://clickhouse.com/docs/en/intro) — armazenamento colunar analítico explicado.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — processamento batch e de stream.
