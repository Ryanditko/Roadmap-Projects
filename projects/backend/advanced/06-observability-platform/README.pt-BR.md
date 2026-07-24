# Plataforma de Observabilidade (logs + métricas)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Quando um sistema distribuído se comporta mal às 3 da manhã, a única coisa entre você e uma noite longa é a sua telemetria. Este projeto pede que você construa a plataforma que a coleta: um pipeline que ingere logs e métricas de muitos serviços, os armazena de forma eficiente sob regras de retenção e permite a um operador consultar, plotar e alertar sobre eles. A parte difícil é o volume e o formato. Logs são texto de alta cardinalidade; métricas são séries temporais compactas — eles exigem motores de armazenamento e consulta diferentes. Você projetará um caminho de ingestão que sobrevive a picos, um store de séries temporais que responde consultas por intervalo rápido, um índice de logs que suporta busca full-text sem explodir em disco e um motor de alertas que dispara em limiares sem afogar o on-call em ruído. Encare isto como construir um Prometheus + Loki em miniatura, não como adotar um.

## Pré-requisitos

- Experiência construindo serviços HTTP/gRPC e workers de background
- Conforto com uma fila de mensagens ou buffer para desacoplar ingestão de armazenamento
- Entendimento de dados de série temporal e agregação (rate, percentil, sum-over-time)
- Familiaridade com conceitos de indexação e busca full-text (índice invertido, tokenização)
- Uma stack de backend de sua escolha além de qualquer datastore que você consiga moldar para séries temporais e texto

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de ingestão desacoplado que absorve picos sem perder dados
- Armazenar métricas como séries temporais e responder consultas de intervalo/agregação eficientemente
- Indexar logs para busca full-text controlando cardinalidade e crescimento em disco
- Construir um motor de regras que avalia condições de alerta em agenda e deduplica
- Impor retenção e downsampling para que o custo de armazenamento fique limitado
- Correlacionar logs e métricas por meio de labels compartilhados e um trace/correlation ID

## Requisitos Funcionais

1. A plataforma deve ingerir logs estruturados e métricas numéricas de múltiplas fontes via um protocolo de push (ou scrape) documentado.
2. A ingestão deve ser desacoplada do armazenamento por um buffer para que uma lentidão de storage não descarte telemetria que chega.
3. Métricas devem ser armazenadas como séries temporais rotuladas e consultáveis por intervalo com agregação (rate, avg, p95, sum).
4. Logs devem ser indexados para busca full-text e por label, com resultados paginados e limitados por tempo.
5. Políticas de retenção devem expirar ou fazer downsample de dados antigos automaticamente por janelas configuradas.
6. Um motor de alertas deve avaliar regras em agenda, disparar quando uma condição se mantém por uma duração e resolver quando ela cessa.
7. Dashboards devem renderizar gráficos de métrica e permitir a um operador ir de um alerta até os logs subjacentes.
8. **Escalabilidade:** o design deve suportar alta vazão de escrita e latência de consulta limitada conforme o volume de dados cresce; documente como o sharding/particionamento por tempo e label funciona.
9. **Confiabilidade:** a ingestão deve degradar graciosamente (buffer, backpressure, shed) em vez de quebrar sob um pico de telemetria.
10. **Observabilidade de si mesma:** a plataforma deve expor seu próprio lag de ingestão, contagem de amostras descartadas e latência de consulta.

## Marcos Sugeridos

1. **Marco 1 — Ingestão e buffering:** Aceite logs e métricas por uma API, empurre-os para uma fila e persista o bruto.
2. **Marco 2 — Store de métricas e consultas:** Armazene séries temporais rotuladas e sirva consultas de intervalo com agregações básicas.
3. **Marco 3 — Índice e busca de logs:** Indexe linhas de log por labels e texto completo; sirva consultas de busca limitadas.
4. **Marco 4 — Alertas e retenção:** Adicione um motor de regras agendado, ciclo de vida de alerta e retenção/downsampling.
5. **Marco 5 — Dashboards e correlação:** Construa gráficos e drill-down de um alerta disparado até os logs correlacionados.

## Esboço de Dados e Interface

```text
Pipeline

  serviços --push--> [ API de Ingestão ] --> [ Buffer/Fila ] --> [ Writers ]
                                                                  /        \
                                              [ TSDB de Métricas ]         [ Índice de Logs ]
                                                     |                            |
                                              [ API de Consulta ] <---- [ Dashboards / Alertas ]
                                                     |
                                              [ Motor de Regras ] --dispara--> [ Notificador ]

Ponto de métrica
  name, labels{service, region, ...}, timestamp, value

Registro de log
  timestamp, level, service, correlationId, message, fields{...}

GET /query/metrics?expr=rate(http_requests_total[5m])&from&to
GET /query/logs?labels={service="api"}&q="timeout"&from&to
POST /rules   { name, expr, forDuration, threshold, notify }
```

## Desafios Extras

- Adicione ingestão de tracing distribuído e visualize a árvore de spans de uma requisição.
- Implemente detecção de anomalias simples (média móvel / z-score) como um tipo de alerta.
- Adicione agrupamento de incidentes para que alertas relacionados colapsem em um único incidente.
- Suporte camadas de downsampling (bruto → 1m → 1h) com roteamento de consulta automático.

## Definição de Pronto

- [ ] Logs e métricas de pelo menos dois serviços fluem pelo buffer até o armazenamento.
- [ ] Uma consulta de intervalo com agregação retorna valores corretos dentro de uma latência limitada.
- [ ] A busca full-text de logs retorna resultados que casam, limitados por tempo e paginados.
- [ ] Um alerta dispara apenas depois que sua condição se mantém pela duração configurada e depois resolve.
- [ ] Dados antigos expiram ou sofrem downsample conforme a política de retenção; o armazenamento não cresce sem limite.
- [ ] Um pico de telemetria é absorvido pelo buffer sem quebrar a ingestão.

## Armadilhas Comuns

- Escrever direto no armazenamento no caminho da requisição — um disco lento então bloqueia todo produtor; bufferize primeiro.
- Deixar a cardinalidade de labels de métrica explodir (labels por usuário ou por requisição), derretendo o store de séries temporais.
- Indexar corpos de log completos sem retenção, de modo que o disco cresce sem limite.
- Alertas que disparam em um único pico em vez de uma condição sustentada, treinando o on-call a ignorá-los.
- Armazenar métricas e logs do mesmo jeito — seus padrões de acesso diferem, e um único motor não serve bem nenhum.

## Recursos

- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — os quatro sinais de ouro e a filosofia de alertas.
- [Prometheus: Modelo de dados e básicos de consulta](https://prometheus.io/docs/concepts/data_model/) — séries temporais rotuladas e PromQL.
- [Grafana Loki: Como funciona](https://grafana.com/docs/loki/latest/get-started/overview/) — indexando logs por labels em vez de conteúdo completo.
- [Documentação do OpenTelemetry](https://opentelemetry.io/docs/) — um padrão neutro de fornecedor para coleta de telemetria.
- [Gorilla: A Fast, Scalable, In-Memory Time Series Database (Facebook)](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf) — o artigo por trás da compressão dos TSDBs modernos.
