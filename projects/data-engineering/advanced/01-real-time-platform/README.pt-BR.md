# Plataforma de Dados em Tempo Real

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa uma plataforma de dados em tempo real de ponta a ponta: eventos brutos chegam por um log durável, um processador de stream os transforma em agregações contínuas, e uma camada de serving de baixa latência responde consultas de dashboards e APIs segundos após o evento chegar. Pense na visão de "métricas ao vivo" por trás de um dashboard de pagamentos ou de uma tela de operações de mobilidade — o valor está inteiramente na atualidade, então um pipeline correto mas dez minutos atrasado falhou. Este projeto força você a raciocinar sobre todo o caminho de uma vez: durabilidade da ingestão, semântica de processamento, estado e latência de leitura. Você escolherá onde aceitar aproximação (contagens em janela) e onde não pode (totais de dinheiro), e projetará para os modos de falha que só aparecem quando os dados nunca param de chegar.

## Pré-requisitos

- Conforto com o modelo central de um processador de stream (Flink, Spark Structured Streaming ou Kafka Streams)
- Experiência operando um log particionado como Kafka ou Pulsar, incluindo grupos de consumidores e offsets
- Uma base em pipelines batch ([ETL distribuído](../02-distributed-etl/) é um bom aquecimento)
- Familiaridade com janelamento, watermarks e event-time vs processing-time

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma topologia ingestão → processamento → serving com garantias de entrega explícitas em cada salto
- Escolher janelamento por event-time e estratégia de watermark para dados fora de ordem e atrasados
- Gerenciar estado grande por chave e raciocinar sobre o custo de checkpoint/restore
- Separar um store de serving quente da camada de processamento e justificar a divisão
- Definir e medir SLOs de latência e atualidade de ponta a ponta

## Requisitos Funcionais

1. A plataforma deve ingerir eventos em um log particionado e reproduzível e sobreviver ao reinício de um broker sem perda de dados.
2. Um job de stream deve calcular agregações por janela de tempo, chaveadas por uma dimensão de negócio (ex.: por comerciante, por região).
3. Eventos atrasados que chegarem dentro de uma tolerância de atraso limitada ainda devem atualizar sua janela; eventos além dela devem ser roteados para uma saída lateral, não descartados silenciosamente.
4. Os resultados devem ser gravados em um store de serving que responda consultas pontuais e por intervalo em milissegundos de um dígito.
5. O sistema deve expor a latência de ponta a ponta (timestamp do evento → consultável) como métrica.
6. Ao reiniciar o job a partir de um checkpoint, as agregações não devem contar em dobro eventos já processados.

## Marcos Sugeridos

1. **Marco 1 — Ingerir e reproduzir:** Suba o log, produza eventos sintéticos com event-time embutido e prove que consegue reproduzir a partir de um offset.
2. **Marco 2 — Processamento em janela:** Implemente janelas por event-time chaveadas, com watermarks, checkpointing e uma saída lateral para dados atrasados.
3. **Marco 3 — Servir e observar:** Envie as agregações ao store quente, adicione uma API de consulta e instrumente SLOs de atualidade e latência.

## Esboço de Dados e Interface

```text
produtores ─▶ [log: tópico "events", N partições, RF=3]
                     │  evento: {id, merchantId, amountCents, eventTime}
                     ▼
              [job de stream]  keyBy(merchantId)
                 janelas tumbling de 1min, watermark = maxEventTime - 30s
                 allowedLateness = 5min ─▶ saída lateral "late"
                 checkpoint a cada 30s ─▶ backend de estado durável
                     │  agg: {merchantId, windowStart, count, sumCents}
                     ▼
              [store quente: Redis / chave-valor]  chave = merchantId:windowStart
                     ▼
GET /metrics/{merchantId}?from=..&to=..  -> [{windowStart, count, sumCents}]
GET /health/freshness                    -> { lagSeconds }
```

## Desafios Extras

- Adicione um segundo caminho de "correção" mais lento (reprocessamento batch) e reconcilie-o com o resultado do streaming — uma comparação lambda/kappa.
- Suporte exactly-once de ponta a ponta usando um sink transacional e chaves idempotentes.
- Adicione auto-scaling do paralelismo de processamento guiado pelo lag do consumidor.

## Definição de Pronto

- [ ] Eventos sobrevivem ao reinício de um broker ou job sem perda e sem contagem dupla.
- [ ] Eventos atrasados dentro do limite atualizam sua janela; além do limite caem na saída lateral.
- [ ] A API de serving retorna agregações em janela para uma chave dentro da latência alvo.
- [ ] O lag de atualidade de ponta a ponta é exportado como métrica e permanece abaixo do SLO declarado sob carga.
- [ ] Um benchmark documentado registra throughput, latência p99 e lag na sua taxa de eventos alvo.

## Armadilhas Comuns

- Misturar semânticas de processing-time e event-time, fazendo os resultados mudarem conforme o momento em que o job roda.
- Definir watermarks agressivos demais e descartar dados legitimamente atrasados, ou frouxos demais e nunca fechar janelas.
- Ignorar o tamanho do estado até que os checkpoints estourem o tempo — chaves ilimitadas crescem para sempre em silêncio.
- Tratar o store de processamento como store de serving, acoplando a latência de leitura aos reinícios do job.

## Recursos

- [Apache Flink: Event Time e Watermarks](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/) — o modelo canônico de tempo em streams.
- [Documentação do Kafka: Design](https://kafka.apache.org/documentation/#design) — como o log te dá durabilidade e replay.
- [The Dataflow Model (artigo)](https://research.google/pubs/pub43864/) — janelamento, watermarks e triggers a partir dos princípios.
- [Guia de Programação do Spark Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) — um modelo de processamento alternativo para comparar.
