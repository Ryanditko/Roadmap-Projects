# Sistema de Monitoramento de Pipelines

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a camada de observabilidade que fica ao lado dos seus pipelines de dados e responde à pergunta que todo engenheiro de dados teme às 9h: "a carga de ontem à noite realmente funcionou?". Você vai capturar cada execução de pipeline como um registro de primeira classe — horário de início, horário de fim, status, contagem de linhas —, avaliá-la contra expectativas de SLA e frescor, e disparar um alerta quando algo estiver atrasado, vazio ou quebrado. O objetivo é pegar uma falha silenciosa antes que um stakeholder pegue. A parte difícil é a calibragem: sensível demais e todos silenciam o canal; frouxo demais e você descobre uma tabela quebrada por um dashboard três dias depois.

## Pré-requisitos

- Um ou dois pipelines cujas execuções você possa instrumentar (mesmo scripts simples agendados)
- Entendimento de SLAs, frescor de dados e a ideia de um limiar de alerta
- Familiaridade com raciocínio de séries temporais (uma métrica observada ao longo de execuções sucessivas)
- Um armazenamento para o histórico de execuções (uma tabela ou coleção de documentos) e algum canal de notificação

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar uma execução de pipeline como um evento estruturado e consultável
- Definir e avaliar regras de SLA e frescor contra o histórico de execuções
- Detectar anomalias como uma queda súbita na contagem de linhas ou um estouro de duração
- Rotear alertas com contexto suficiente para agir e suprimir ruído duplicado
- Apresentar a saúde dos pipelines de relance para alguém que não está de plantão

## Requisitos Funcionais

1. Toda execução de pipeline deve ser registrada com horário de início, horário de fim, status e linhas processadas.
2. O sistema deve sinalizar uma execução que falha, excede sua duração esperada ou produz zero linhas.
3. O sistema deve avaliar o frescor dos dados — alertar quando um dataset não é atualizado dentro de sua janela de SLA.
4. O sistema deve detectar uma anomalia de contagem de linhas relativa a uma baseline móvel (ex.: queda > 50%).
5. Um alerta deve incluir o nome do pipeline, qual regra disparou, o valor observado e o valor esperado.
6. Alertas repetidos para a mesma falha em andamento devem ser deduplicados, não reenviados a cada execução.
7. O sistema deve expor uma visão de saúde listando a última execução, o status e o frescor de cada pipeline.

## Marcos Sugeridos

1. **Marco 1 — Capturar execuções:** Instrumente os pipelines para emitir um registro de execução e armazene o histórico.
2. **Marco 2 — Regras e alertas:** Adicione regras de SLA, frescor e contagem de linhas que produzam alertas com contexto.
3. **Marco 3 — Controle de ruído e dashboard:** Deduplique alertas e construa uma visão de saúde sobre o histórico de execuções.

## Esboço de Dados e Interface

```text
pipeline_run
  run_id        uuid
  pipeline      string
  started_at    timestamp
  ended_at      timestamp
  status        enum(success | failed | running)
  rows_out      integer

tipos de regra:
  sla_duration   -> ended_at - started_at > limiar
  freshness      -> now - last_success.ended_at > janela
  volume_anomaly -> rows_out < baseline * (1 - drop_pct)

alert
  pipeline, rule, observed, expected, first_seen, still_open
  -> notificar em <canal>; deduplicar enquanto still_open

visão de saúde: pipeline | last_run | status | age | open_alerts
```

## Desafios Extras

- Adicione pistas de causa raiz correlacionando uma falha com o pipeline upstream que o alimenta.
- Rastreie um trace por execução para que uma etapa lenta dentro de um pipeline fique visível, não apenas a duração total.
- Suporte severidades de alerta e escalonamento (avisar no chat, acionar plantão em crítico repetido).
- Resolva um alerta automaticamente e poste um aviso de recuperação quando a próxima execução for bem-sucedida.

## Definição de Pronto

- [ ] Toda execução aparece no histórico com status e contagem de linhas corretos.
- [ ] Uma execução com falha ou vazia produz exatamente um alerta, não um por nova tentativa.
- [ ] Um dataset que fica desatualizado além de seu SLA dispara um alerta de frescor sem que uma execução falhe.
- [ ] Uma queda súbita na contagem de linhas é sinalizada contra a baseline móvel.
- [ ] A visão de saúde reflete o verdadeiro estado da última execução de cada pipeline.

## Armadilhas Comuns

- Alertar a cada execução até o canal virar ruído que todos ignoram — deduplique incidentes abertos.
- Medir frescor a partir do início da execução em vez da conclusão bem-sucedida, escondendo falhas parciais.
- Usar um limiar fixo de contagem de linhas que quebra com tráfego sazonal; prefira uma baseline móvel.
- Enviar alertas sem contexto, forçando o leitor a investigar o que de fato quebrou.
- Tratar um job longo ainda em execução como uma falha porque você só verifica um registro concluído.

## Recursos

- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — os quatro sinais de ouro e a filosofia de alertas.
- [Prometheus: Boas práticas de alerta](https://prometheus.io/docs/practices/alerting/) — como escrever regras que permanecem acionáveis.
- [Airflow: SLAs e monitoramento](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html#slas) — conceitos de SLA em um orquestrador real.
- [Monte Carlo: O que é observabilidade de dados](https://www.montecarlodata.com/blog-what-is-data-observability/) — frescor, volume e schema como pilares de observabilidade.
