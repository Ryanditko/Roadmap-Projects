# Projete um Sistema de Métricas e Monitoramento

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um sistema que coleta métricas numéricas de serviços — taxas de requisição, latências, CPU — as armazena como séries temporais e permite consultá-las e alertar sobre elas. Pense em um Prometheus mais Grafana simplificado. Ao contrário de logs, métricas são pequenas, regulares e numéricas, o que convida à agregação: você raramente precisa de cada ponto bruto para sempre, então faz downsampling dos dados antigos. O risco dominante é a explosão de cardinalidade, em que combinações demais de labels estouram o armazenamento. Você vai raciocinar sobre o modelo de coleta, o armazenamento de séries temporais e os rollups. Entregue um documento de design cobrindo ingestão, armazenamento e consulta.

## Pré-requisitos

- Entendimento do que é uma métrica (um número amostrado ao longo do tempo)
- Consciência dos tipos de métrica: counters, gauges, histograms
- Familiaridade com a ideia de uma série temporal (um fluxo de valores identificado por labels)
- Conforto para raciocinar sobre agregação em janelas de tempo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um modelo de coleta de métricas (push vs. pull) e justificá-lo
- Raciocinar sobre armazenamento de séries temporais e por que difere de um banco genérico
- Entender cardinalidade e como labels podem estourar o armazenamento
- Projetar downsampling e retenção para limitar custo
- Enunciar um trade-off entre resolução e custo de armazenamento

## Requisitos e Restrições

1. Coletar métricas de muitos serviços em um intervalo regular.
2. Armazená-las como séries temporais identificadas por um nome mais labels.
3. Consultar por nome, filtros de label e intervalo de tempo, com agregação (avg, p99, rate).
4. Fazer downsampling de dados antigos para manter o armazenamento limitado.
5. Suportar alertas quando uma métrica cruza um limiar.
6. Estime a escala: 1M séries temporais ativas, coletadas a cada 15s — cerca de 67K amostras/s.

## Abordagem Sugerida

1. Escolha coleta push vs. pull e note o trade-off (pull dá controle ao servidor; push serve jobs de vida curta).
2. Faça a conta: 1M séries ÷ 15s ≈ 67K amostras/s; cada amostra é minúscula, mas a contagem de séries dita memória e tamanho do índice.
3. Projete a identidade da série: nome da métrica + conjunto ordenado de labels = uma série; raciocine sobre como labels multiplicam a cardinalidade.
4. Projete um armazenamento que anexa amostras por série e faz downsampling de janelas mais antigas.
5. Esboce o caminho de consulta e alerta: avaliar uma regra sobre dados recentes de forma agendada.

## Esboço de Arquitetura

```text
Serviços ── expõem /metrics ──> [ Scraper ] ──> [ Store de Séries Temporais ]
     (pull, a cada 15s)                               │  downsample janelas antigas
                                                [ Store de Rollup / Longo Prazo ]
Engenheiro ── query ──> [ Motor de Consulta ] ──> série   [ Alerter ] --regra--> notifica

API principal
  GET  /metrics                (exposição, por serviço)
  GET  /query   ?expr=&from=&to=&step=   -> séries temporais
  POST /alerts  { expr, threshold, for } -> regra de alerta

Modelo de dados
  id de série = metric_name + {label1=v1,...}   (labels ditam cardinalidade)
  amostra: series_id | ts | value
  retenção: bruto 15s por 7d -> rollup 5m por 90d -> rollup 1h por 1a
```

## Tópicos de Aprofundamento

- **Explosão de cardinalidade:** como um label de alta cardinalidade (como ID de usuário) multiplica séries em milhões e arruína o armazenamento.
- **Push vs. pull:** descoberta de serviço e sinal de saúde com pull, vs. push para jobs batch efêmeros.
- **Downsampling:** agregar amostras brutas em rollups mais grossos para que um ano de histórico continue acessível.

## Entregáveis

- Um diagrama de arquitetura mostrando scraping, armazenamento de séries temporais, consulta e alerta.
- O contrato da API de consulta e regra de alerta.
- Um modelo de dados para identidade de série, amostras e o esquema de retenção/rollup.
- Um trade-off descrito: ex., alta resolução com longa retenção (histórico rico, armazenamento caro) vs. downsampling agressivo (barato, mas dados antigos perdem detalhe fino).

## Armadilhas Comuns

- Adicionar um label ilimitado (ID de usuário, ID de requisição) e detonar a cardinalidade.
- Armazenar métricas em um BD relacional genérico e esbarrar em limites de throughput de escrita e consulta.
- Manter dados em resolução total para sempre em vez de fazer downsampling, de modo que o custo cresce sem limite.
- Confundir métricas com logs — métricas são números agregáveis, não texto buscável.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de armazenamento e escalabilidade.
- [Prometheus: Visão geral](https://prometheus.io/docs/introduction/overview/) — o sistema de métricas pull-based de referência.
- [Prometheus: Modelo de dados](https://prometheus.io/docs/concepts/data_model/) — séries, labels e cardinalidade.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — o que medir e sobre o que alertar.
