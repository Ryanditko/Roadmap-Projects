# Stack de Monitoramento (Prometheus + Grafana)

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Suba uma stack de observabilidade baseada em métricas: instrumente uma aplicação para expor métricas, faça o Prometheus coletá-las e armazená-las como séries temporais, visualize-as no Grafana e roteie alertas pelo Alertmanager quando algo cruzar um limite. Este é o projeto de "saber que seu sistema está saudável antes dos seus usuários avisarem". Você vai passar de contadores brutos para sinais significativos — taxa de requisições, taxa de erros e latência (o método RED) — e aprender por que alertar sobre sintomas é melhor do que alertar sobre causas.

## Pré-requisitos

- Uma aplicação que você possa instrumentar (ou que já exponha métricas Prometheus)
- Docker / Docker Compose ou um cluster para rodar Prometheus, Grafana e Alertmanager
- Entendimento de counters, gauges e histograms como tipos de métrica
- Familiaridade básica com consultas e dashboards

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Instrumentar uma app com counters, gauges e histograms expostos em um endpoint `/metrics`
- Configurar o Prometheus para descobrir e coletar alvos
- Escrever consultas PromQL para taxa, proporção de erros e percentis de latência
- Construir um dashboard no Grafana a partir dessas consultas
- Definir regras de alerta e rotear notificações pelo Alertmanager

## Requisitos Funcionais

1. A app alvo deve expor métricas no formato de exposição do Prometheus em um endpoint HTTP.
2. O Prometheus deve coletar o alvo em um intervalo e armazenar as séries.
3. A app deve expor ao menos taxa de requisições, contagem de erros e histograma de duração de requisição.
4. Um dashboard no Grafana deve mostrar os sinais RED (Taxa, Erros, Duração) ao longo do tempo.
5. Ao menos uma regra de alerta deve ser definida (ex.: proporção de erros acima de um limite por N minutos).
6. O Alertmanager deve entregar um alerta disparado a um canal de notificação.
7. Regras de recording ou de alerta devem usar `rate()`/`histogram_quantile()` corretamente sobre uma janela de tempo.

## Marcos Sugeridos

1. **Marco 1 — Instrumentar e coletar:** Exponha `/metrics`, faça o Prometheus coletá-lo.
2. **Marco 2 — Visualizar:** Escreva PromQL para os sinais RED e construa um dashboard no Grafana.
3. **Marco 3 — Alertar:** Adicione uma regra de alerta e roteie-a pelo Alertmanager até um canal.

## Esboço de Dados e Interface

```text
Fluxo:
  app /metrics ──coleta──> Prometheus (TSDB) ──consulta(PromQL)──> dashboards Grafana
                                 |
                          avalia regras de alerta ─> Alertmanager ─> notifica (email/Slack/webhook)

Tipos de métrica em /metrics:
  http_requests_total{method,status}      counter
  http_request_duration_seconds           histogram (buckets)
  process_resident_memory_bytes           gauge

Consultas RED (PromQL conceitual):
  taxa:     sum(rate(http_requests_total[5m]))
  erros:    sum(rate(http_requests_total{status=~"5.."}[5m]))
  duração:  histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

## Desafios Extras

- Adicione exporters (node_exporter, cAdvisor) para métricas de host/contêiner ao lado das da app.
- Adicione recording rules para pré-computar consultas caras usadas pelos dashboards.
- Configure roteamento de alertas com severidades, agrupamento e silences no Alertmanager.
- Provisione dashboards e datasources como código para que a stack seja reproduzível.

## Definição de Pronto

- [ ] O Prometheus mostra o alvo como "up" e armazena suas séries.
- [ ] Um dashboard no Grafana exibe taxa, proporção de erros e latência p95.
- [ ] O PromQL usa `rate()` sobre um intervalo, não valores brutos do counter.
- [ ] Um alerta transiciona para disparado quando a condição se mantém e chega a um canal.
- [ ] Reiniciar a app não produz alertas falsos por resets de counter.

## Armadilhas Comuns

- Plotar um counter bruto em vez do seu `rate()`, produzindo uma linha sempre crescente e sem sentido.
- Alertar a cada oscilação transitória sem uma duração `for:`, criando fadiga de alertas.
- Labels de alta cardinalidade (id de usuário, id de requisição) explodindo a memória do Prometheus.
- Interpretar mal percentis de histograma aplicando `histogram_quantile` sem `rate()` nos buckets.
- Alertar sobre causas (CPU alta) em vez de sintomas (usuários vendo erros), fazendo os pages não mapearem para impacto.

## Recursos

- [Documentação do Prometheus](https://prometheus.io/docs/introduction/overview/) — modelo de dados, coleta e PromQL.
- [Documentação do Grafana](https://grafana.com/docs/grafana/latest/) — dashboards e fontes de dados.
- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) — roteamento, agrupamento e silêncio de alertas.
- [Livro de SRE do Google: Monitorando Sistemas Distribuídos](https://sre.google/sre-book/monitoring-distributed-systems/) — sinais que valem alerta.
</content>
