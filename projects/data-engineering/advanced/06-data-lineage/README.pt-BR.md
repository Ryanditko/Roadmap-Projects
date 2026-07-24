# Sistema de Linhagem de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um sistema que rastreia a linhagem de dados — de quais datasets uma dada tabela foi derivada, por quais jobs e (idealmente) no nível de coluna. Quando um dashboard a jusante mostra números errados, a linhagem responde "o que alimenta isto?" em segundos em vez de um dia de arqueologia; quando uma coluna de origem muda, a linhagem responde "o que quebra?" antes de você subir. Você capturará linhagem automaticamente de jobs em execução (emitindo eventos de run/dataset/job, idealmente no formato OpenLineage), a armazenará como um grafo acíclico direcionado, e exporá consultas para travessia a montante/jusante e análise de impacto. Os desafios centrais são *completude* (um job que você não instrumenta é um ponto cego), manter as consultas de grafo rápidas conforme o grafo cresce, e capturar linhagem sem pedir aos engenheiros que a mantenham à mão.

## Pré-requisitos

- Familiaridade com pipelines/DAGs e um orquestrador (Airflow, Dagster ou similar)
- Conforto com modelagem de grafos e um store de grafo ou relacional
- Entendimento de jobs produzindo/consumindo datasets
- Conhecimento básico de um padrão de linhagem como OpenLineage (útil, não obrigatório)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar linhagem como um DAG de datasets, jobs e runs
- Capturar linhagem automaticamente da execução dos jobs em vez de anotação manual
- Suportar travessia a montante ("o que alimenta X") e a jusante ("o que depende de X")
- Rodar análise de impacto: dada uma origem alterada/quebrada, listar tudo que é afetado
- Raciocinar sobre lacunas de completude e como a linhagem parcial engana

## Requisitos Funcionais

1. Todo run de job deve emitir um evento de linhagem registrando seus datasets de entrada, de saída e o status do run.
2. O sistema deve armazenar linhagem como um DAG consultável com nós para datasets e jobs.
3. Uma consulta deve retornar toda a cadeia a montante de qualquer dataset (fontes transitivas).
4. Uma consulta deve retornar toda a cadeia a jusante (consumidores transitivos) para análise de impacto.
5. A linhagem no nível de coluna deve ser suportada para ao menos uma transformação (quais colunas de entrada produziram uma coluna de saída).
6. O sistema deve registrar mudanças de schema/versão nos datasets ao longo do tempo.

## Marcos Sugeridos

1. **Marco 1 — Captura:** Emita eventos de run/job/dataset de alguns passos do pipeline e persista-os.
2. **Marco 2 — Grafo e travessia:** Construa o DAG e exponha consultas a montante/jusante com proteção contra ciclos.
3. **Marco 3 — Impacto e colunas:** Adicione linhagem no nível de coluna para uma transformação e uma consulta de análise de impacto, mais uma visualização de linhagem.

## Esboço de Dados e Interface

```text
evento de linhagem (por run):
  { runId, job: "etl.orders_daily", state: complete,
    inputs:  [orders_raw, fx_rates],
    outputs: [orders_daily],
    columnLineage: { "orders_daily.amount_usd":
                       ["orders_raw.amount", "fx_rates.rate"] } }

grafo:
  orders_raw ─▶ [etl.orders_daily] ─▶ orders_daily ─▶ [bi.revenue] ─▶ revenue_dash
  fx_rates  ─┘

consultas:
  upstream(revenue_dash)   -> {orders_daily, orders_raw, fx_rates}
  downstream(orders_raw)   -> {orders_daily, revenue_dash}   (análise de impacto)
  columnsFor(orders_daily.amount_usd) -> [orders_raw.amount, fx_rates.rate]

Não funcional: travessia p95 < T em N nós; captura adiciona < X% de overhead ao job;
completude = jobs instrumentados / jobs totais (acompanhe isso).
```

## Desafios Extras

- Integre o OpenLineage para que a linhagem seja emitida por conectores reais, não apenas pelos seus hooks.
- Adicione "propagação de atualidade": marque todos os datasets a jusante como obsoletos quando um run a montante falhar.
- Detecte e alerte sobre datasets órfãos (sem job produtor) e fontes sem saída.

## Definição de Pronto

- [ ] Runs de jobs emitem eventos de linhagem automaticamente; nenhuma edição manual de grafo é necessária.
- [ ] Travessias a montante e a jusante retornam conjuntos transitivos corretos, seguras contra ciclos.
- [ ] A linhagem no nível de coluna está correta para ao menos uma transformação não trivial.
- [ ] A análise de impacto lista tudo afetado por uma origem alterada.
- [ ] Uma visualização renderiza o DAG e a travessia p95 permanece dentro do alvo conforme o grafo cresce.

## Armadilhas Comuns

- Linhagem manual que se descola no momento em que um job muda — a captura automática é o ponto central.
- Lacunas de completude silenciosas: um job não instrumentado quebra a cadeia e ninguém percebe.
- Armazenar linhagem sem versionamento por run/tempo, então você não consegue responder "como isto estava terça passada".
- Travessias ilimitadas sem detecção de ciclos, travando em um grafo em forma de diamante.

## Recursos

- [Documentação do OpenLineage](https://openlineage.io/docs/) — o padrão aberto para eventos de linhagem.
- [OpenLineage: Linhagem no nível de coluna](https://openlineage.io/docs/spec/facets/dataset-facets/column_lineage_facet/) — o facet que modela a derivação de colunas.
- [Marquez](https://marquezproject.github.io/marquez/) — um serviço de metadados de linhagem de referência construído sobre OpenLineage.
- [Apache Atlas](https://atlas.apache.org/#/) — um sistema corporativo de metadados e governança de linhagem para comparar.
