# Plataforma Completa de Observabilidade

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa a plataforma que responde "o que está acontecendo em produção, e por quê?" unificando os três pilares — métricas, logs e traces — atrás de uma história de correlação que permite ao operador saltar de um dashboard disparando para o trace exato e para a linha de log específica. Você vai instrumentar serviços, enviar telemetria por um coletor, armazenar cada sinal em um backend apropriado e amarrá-los com identificadores compartilhados para que uma única requisição seja seguida de ponta a ponta. Os desafios avançados são os que decidem se isso sobrevive ao contato com tráfego real: controle de cardinalidade nas métricas, amostragem nos traces, retenção e custo em volume, e alertas que acionam um humano apenas quando um humano é de fato necessário. Uma boa plataforma de observabilidade transforma depuração de arqueologia em uma query.

## Pré-requisitos

- Um ou mais serviços em execução que você possa instrumentar (idealmente com um caminho de requisição entre serviços)
- Familiaridade com métricas no estilo Prometheus e o básico de PromQL
- Entendimento de logging estruturado e conceitos de tracing distribuído
- Conforto para implantar backends com estado e um coletor de telemetria

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Instrumentar serviços para métricas, logs e traces usando padrões abertos (OpenTelemetry)
- Correlacionar os três sinais via IDs de trace/span e labels consistentes
- Controlar cardinalidade, amostragem e retenção para manter custo e volume sob controle
- Construir dashboards e alertas guiados por SLIs, não por métricas de vaidade
- Projetar alertas que minimizam falsos acionamentos enquanto pegam degradação real

## Requisitos Funcionais

1. Serviços devem emitir métricas, logs estruturados e traces distribuídos por um pipeline comum.
2. Uma única requisição deve ser rastreável entre pelo menos dois serviços com um trace ID compartilhado.
3. Um dashboard deve apresentar SLIs (latência, taxa de erro, saturação) para cada serviço-chave.
4. Logs devem ser pesquisáveis e vinculáveis a um trace por um identificador de correlação.
5. Alertas devem disparar em queima de SLO, não em limiares brutos, e rotear para um canal de notificação.
6. A plataforma deve aplicar amostragem e/ou limites de cardinalidade para limitar o custo.
7. Políticas de retenção devem ser definidas por sinal e impostas.

## Marcos Sugeridos

1. **Marco 1 — Métricas e dashboards:** Colete métricas, defina SLIs e construa um dashboard de serviço.
2. **Marco 2 — Logs e tracing:** Adicione logs estruturados e tracing distribuído através de um coletor.
3. **Marco 3 — Correlação:** Conecte trace IDs aos logs para poder pivotar de sinal para sinal.
4. **Marco 4 — Alertas e controle de custo:** Adicione alertas de queima de SLO, amostragem, limites de cardinalidade e políticas de retenção.

## Esboço de Dados e Interface

```text
   serviços (SDK OTel)
      │ métricas   │ logs        │ traces
      ▼            ▼             ▼
   ┌──────────────────────────────────┐
   │   Coletor OpenTelemetry          │  (batch, amostra, enriquece)
   └───────┬──────────┬──────────┬────┘
           ▼          ▼          ▼
     ┌──────────┐ ┌────────┐ ┌────────┐
     │Prometheus│ │  Loki  │ │ Tempo/ │
     │ (métricas)│ │ (logs) │ │ Jaeger │
     └────┬─────┘ └───┬────┘ └───┬────┘
          └──────┬────┴──────────┘
                 ▼    correlaciona por trace_id + labels
           ┌───────────┐        ┌────────────┐
           │  Grafana  │        │ Alertmanager│──▶ notifica
           │ dashboards│        │(queima SLO)│
           └───────────┘        └────────────┘

Contrato de correlação:
  toda linha de log carrega: trace_id, span_id, service, level
  toda métrica carrega:      service, route, status (conjunto limitado de labels)

Metas não-funcionais:
  cardinalidade de métrica  limitada (< N séries por serviço)
  amostragem de trace       head ou tail sampling para limitar volume
  precisão de alerta        aciona só em queima de SLO; taxa de ruído monitorada
```

## Desafios Extras

- Adicione exemplars ligando um pico de métrica diretamente a um trace representativo.
- Introduza detecção de anomalias ou alertas de SLO por múltiplas janelas e taxas de queima.
- Adicione isolamento por tenant/time para que custo e acesso sejam atribuíveis.
- Construa um fluxo de "um clique do alerta à causa raiz" atravessando os três sinais.

## Definição de Pronto

- [ ] Os três sinais fluem por um único pipeline de coletor.
- [ ] Uma requisição é seguida de ponta a ponta entre serviços via um trace ID.
- [ ] Uma linha de log pode ser pivotada para seu trace e vice-versa.
- [ ] Alertas disparam em queima de SLO e roteiam para um canal real, com falsos acionamentos minimizados.
- [ ] Amostragem, limites de cardinalidade e retenção estão configurados e impostos.

## Armadilhas Comuns

- Cardinalidade de label ilimitada (IDs de usuário, URLs) que explode o backend de métricas e o custo.
- Coletar os três sinais mas nunca correlacioná-los, então depurar ainda significa três abas.
- Alertas por limiar que acionam às 3h por uma oscilação transitória sobre a qual ninguém precisa agir.
- Nenhuma amostragem nos traces, então o custo de ingestão e armazenamento cresce linearmente com o tráfego para sempre.
- Tratar dashboards como o produto; o produto são respostas rápidas, que precisam de SLIs e correlação.

## Recursos

- [Documentação do OpenTelemetry](https://opentelemetry.io/docs/) — padrão de instrumentação neutro em relação a fornecedor.
- [Documentação do Prometheus](https://prometheus.io/docs/) — coleta de métricas e PromQL.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — os quatro sinais de ouro.
- [Google SRE Workbook: Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) — alertas por taxa de queima bem feitos.
