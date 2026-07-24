# Carregador de Data Warehouse

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa a camada de carga que transforma registros operacionais brutos em um modelo dimensional amigável para consultas em um warehouse. Você vai aterrissar os dados de origem em uma área de staging e depois carregá-los em tabelas fato e dimensão modeladas como um star schema. A peça central é lidar com *dimensões que mudam lentamente* (SCD Tipo 2): quando um cliente muda de cidade, você mantém a linha antiga, a encerra e abre uma nova linha versionada — para que fatos históricos ainda façam join com o endereço que era verdadeiro na época. Você também tornará a carga incremental e idempotente, para que reprocessar um lote se corrija em vez de dobrar a receita, e suportará um backfill que reconstrói um intervalo de lotes passados. Essa é a diferença entre um warehouse em que as pessoas confiam e uma planilha com passos extras.

## Pré-requisitos

- SQL sólido: joins, `GROUP BY`, funções de janela e semântica de `MERGE`/upsert
- Entendimento de chaves primárias/estrangeiras e integridade referencial
- Familiaridade com um warehouse ou banco (Postgres, DuckDB, BigQuery ou Snowflake)
- Noção básica da distinção ETL/ELT
- O [projeto de ETL com Airflow](../01-airflow-etl/) é um bom aquecimento para a orquestração da carga

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um star schema com tabelas fato e dimensão e chaves substitutas
- Implementar SCD Tipo 2 para que o histórico da dimensão seja preservado e consultável em uma data
- Carregar incrementalmente usando uma marca d'água em vez de reprocessar tudo
- Tornar cargas idempotentes via upsert/merge para que reprocessamentos nunca dupliquem contagens
- Fazer backfill de um intervalo de lotes e verificar a integridade referencial depois

## Requisitos Funcionais

1. Os dados de origem devem primeiro aterrissar em uma tabela de staging antes de qualquer transformação no modelo.
2. O warehouse deve expor ao menos uma tabela fato e duas dimensões em star schema, chaveadas por chaves substitutas.
3. Uma dimensão rastreada deve implementar SCD Tipo 2 com `valid_from`, `valid_to` e um flag de linha atual.
4. As cargas devem ser incrementais, selecionando apenas linhas alteradas desde a última marca d'água bem-sucedida.
5. Reprocessar um lote deve ser idempotente — um upsert/merge, não um insert cego.
6. Linhas de fato devem referenciar a versão da dimensão que era atual no timestamp do evento.
7. Um backfill deve conseguir reconstruir um intervalo especificado de lotes sem duplicar linhas.

## Marcos Sugeridos

1. **Marco 1 — Staging e carga de dimensões:** Aterrisse a origem em staging e popule as dimensões com chaves substitutas.
2. **Marco 2 — SCD Tipo 2 e fatos:** Versione uma dimensão que muda e carregue fatos que fazem join com a versão correta.
3. **Marco 3 — Incremental e idempotente:** Adicione uma marca d'água, torne o merge idempotente e teste um backfill.

## Esboço de Dados e Interface

```text
staging_customer(cols brutas..., loaded_at)

dim_customer                          fact_orders
  customer_sk   PK (substituta)         order_sk     PK
  customer_id   (chave natural/negócio) customer_sk  FK -> dim_customer
  city                                  order_date_sk FK -> dim_date
  valid_from    timestamp               amount
  valid_to      timestamp (ou NULL)     ...
  is_current    boolean

SCD2 na mudança de city:
  UPDATE dim_customer SET valid_to = now(), is_current = false
    WHERE customer_id = ? AND is_current;
  INSERT nova linha com nova city, valid_from = now(), is_current = true;

Incremental: WHERE source.updated_at > ultima_marca_dagua
Carga idempotente: MERGE ... ON chave_natural WHEN MATCHED ... WHEN NOT MATCHED ...
```

## Desafios Extras

- Adicione SCD Tipo 1 para um atributo cujo histórico não vale a pena manter, e contraste os dois.
- Construa uma consulta "as-of" que reconstrói o estado da dimensão em uma data passada arbitrária.
- Adicione um tratador de fato tardio que retroage à versão correta da dimensão.
- Rastreie métricas de carga (linhas inseridas/atualizadas/rejeitadas) por lote para monitoramento.

## Definição de Pronto

- [ ] Fatos fazem join com dimensões sem nenhuma chave substituta órfã (a integridade referencial se mantém).
- [ ] Mudar um atributo rastreado encerra a linha antiga da dimensão e abre uma nova atual.
- [ ] Reprocessar o mesmo lote mantém as contagens de linhas inalteradas (merge idempotente verificado).
- [ ] Um run incremental toca apenas linhas alteradas desde a última marca d'água.
- [ ] Um backfill de vários lotes reproduz o mesmo resultado de carregá-los em ordem.

## Armadilhas Comuns

- Usar a chave de negócio natural como FK do fato, o que quebra no momento em que o SCD2 cria uma segunda versão.
- Esquecer de encerrar o `valid_to` da linha anterior, deixando duas linhas "atuais" para uma entidade.
- Cargas com `INSERT` cego que duplicam dados no reprocessamento em vez de `MERGE`/upsert.
- Avançar a marca d'água antes de a carga confirmar, fazendo um crash no meio pular linhas para sempre.
- Intervalos `valid_from`/`valid_to` sobrepostos que fazem joins "as-of" retornarem duplicatas.

## Recursos

- [Kimball Group: Técnicas de Modelagem Dimensional](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/) — a referência canônica de star schemas e SCDs.
- [Wikipedia: Slowly Changing Dimension](https://en.wikipedia.org/wiki/Slowly_changing_dimension) — um tour conciso pelos tipos 1–6 de SCD.
- [dbt: snapshots / SCD](https://docs.getdbt.com/docs/build/snapshots) — como uma ferramenta moderna modela histórico Tipo 2.
- [Star schema (Wikipedia)](https://en.wikipedia.org/wiki/Star_schema) — fatos, dimensões e chaves substitutas.
