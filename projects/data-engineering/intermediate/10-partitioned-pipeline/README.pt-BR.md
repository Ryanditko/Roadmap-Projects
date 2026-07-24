# Pipeline de Dados Particionado

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um pipeline que escreve sua saída dividida em partições — tipicamente por data — para que as leituras varram apenas as fatias necessárias e o reprocessamento toque uma partição em vez da tabela inteira. Esse é o layout sob todo grande data lake: pastas `dt=2026-07-24/`, poda de partição no momento da consulta e retenção por partição. Você vai escolher uma chave de partição, escrever dados na partição certa, tornar cada partição reconstruível de forma independente e provar que as consultas podam as partições de que não precisam. O ganho é concreto: um backfill ou uma correção de um dia ruim vira uma operação de uma partição em vez de uma recarga completa.

## Pré-requisitos

- Conforto para escrever um job em lote que produz um dataset (arquivos ou tabelas de warehouse)
- Entendimento de como a poda de partição acelera consultas
- Familiaridade com escritas idempotentes e a ideia de carga incremental ([Processamento Incremental de Dados](../06-incremental-processing/) é um bom aquecimento)
- Um formato colunar ou destino de tabela particionada (Parquet, estilo Delta/Iceberg/Hive ou uma tabela SQL particionada)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher uma chave de partição pelo padrão de consulta e cardinalidade, não por hábito
- Escrever dados na partição correta e sobrescrever uma única partição de forma idempotente
- Verificar que uma consulta filtrada poda apenas as partições relevantes
- Gerenciar o ciclo de vida de uma partição: retenção, compactação e limpeza
- Reprocessar ou reparar uma partição sem perturbar as demais

## Requisitos Funcionais

1. O pipeline deve escrever a saída particionada por uma chave escolhida (ex.: data do evento).
2. Reexecutar para uma dada partição deve sobrescrever totalmente aquela partição, não anexar duplicatas.
3. Uma consulta filtrada pela chave de partição deve ler apenas as partições correspondentes (layout podável).
4. O pipeline deve suportar o backfill de um conjunto arbitrário de partições passadas.
5. Uma política de retenção deve descartar partições mais antigas que um horizonte configurado.
6. O layout deve evitar a explosão de arquivos minúsculos — controle o número de arquivos por partição.
7. O pipeline deve registrar quais partições foram escritas ou reconstruídas em cada execução.

## Marcos Sugeridos

1. **Marco 1 — Escrita particionada:** Escolha uma chave e escreva a saída em caminhos por partição.
2. **Marco 2 — Sobrescrita idempotente e poda:** Sobrescreva uma única partição na reexecução e confirme que as consultas podam.
3. **Marco 3 — Ciclo de vida:** Adicione backfill, retenção e controle da contagem de arquivos.

## Esboço de Dados e Interface

```text
layout de partições (particionado por data):
  /warehouse/events/
    dt=2026-07-22/  part-000.parquet ...
    dt=2026-07-23/  part-000.parquet ...
    dt=2026-07-24/  part-000.parquet ...

contrato de escrita:
  write(partition=dt, rows) -> sobrescrever APENAS o diretório daquele dt
  backfill(dt_from, dt_to)  -> reconstruir cada dt no intervalo de forma independente

verificação de poda:
  query WHERE dt = '2026-07-24'  -> lê 1 partição, não a tabela inteira

ciclo de vida:
  retenção:    descartar dt < hoje - N dias
  compactação: mesclar muitos arquivos pequenos -> poucos arquivos do tamanho alvo
  run_manifest: run_id | partitions_written[] | files | rows
```

## Desafios Extras

- Trate o skew: detecte uma partição quente e a subparticione (ex.: por bucket de hash) para equilibrar a carga.
- Adicione particionamento multinível (dt + region) e raciocine sobre o tradeoff de explosão de partições.
- Rastreie estatísticas em nível de partição para que um planejador de consultas pule via metadados min/max, não só pela chave.
- Suporte dados atrasados reabrindo e reescrevendo uma partição já fechada de forma idempotente.

## Definição de Pronto

- [ ] A saída chega no caminho de partição correto para sua chave.
- [ ] Reprocessar uma data sobrescreve exatamente aquela partição, deixando as outras intactas.
- [ ] Uma consulta filtrada por partição comprovadamente varre apenas as partições correspondentes.
- [ ] Reprocessar um intervalo de datas reconstrói cada partição de forma independente.
- [ ] A retenção remove partições expiradas e a contagem de arquivos permanece limitada por partição.

## Armadilhas Comuns

- Escolher uma chave de partição de alta cardinalidade (ex.: user_id) e criar milhões de partições minúsculas.
- Anexar na reexecução, de modo que um dia reprocessado silenciosamente dobra suas linhas.
- Particionar por uma coluna pela qual as consultas nunca filtram, então a poda nunca acontece.
- O problema dos arquivos pequenos: milhares de arquivos de KB por partição destruindo o desempenho de leitura.
- Apagar os arquivos de uma partição antes de a nova escrita chegar, deixando uma lacuna em caso de falha.

## Recursos

- [Apache Hive: Tabelas particionadas](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-PartitionedTables) — a convenção original de partição por diretório.
- [Databricks: Boas práticas de particionamento](https://docs.databricks.com/aws/en/tables/partitions) — quando particionar e a armadilha dos arquivos pequenos.
- [Apache Iceberg: Particionamento](https://iceberg.apache.org/docs/latest/partitioning/) — particionamento oculto e evolução feitos direito.
- [Spark: Fontes de dados — descoberta de partições](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html#partition-discovery) — como a poda usa o layout.
