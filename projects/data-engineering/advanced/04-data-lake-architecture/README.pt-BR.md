# Arquitetura de Data Lake Escalável

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete um data lake escalável que se comporta como um warehouse onde importa — um **lakehouse**. Arquivos brutos no armazenamento de objetos são baratos e infinitos, mas consultam devagar e não fazem updates atômicos; um formato de tabela (Apache Iceberg, Delta Lake ou Apache Hudi) fica por cima e adiciona commits ACID, evolução de schema, time travel e compactação de arquivos. Você projetará o layout de armazenamento (tiering quente/morno/frio, particionamento, tamanho de arquivo), escolherá um formato de tabela e o justificará, e provará que um lake bem organizado responde consultas analíticas rápido enquanto um despejo ingênuo de arquivos pequenos não. O tema que atravessa tudo é que layout *é* performance: partition pruning, compactação de arquivos e pruning de metadados são o que separa uma consulta que varre um terabyte de uma que varre um gigabyte.

## Pré-requisitos

- Conforto com armazenamento de objetos (S3/GCS/Azure Blob) e um formato colunar (Parquet/ORC)
- Um motor de consulta que você possa apontar para arquivos (Spark, Trino, DuckDB ou Athena)
- Entendimento de particionamento e predicate pushdown
- Familiaridade com o "problema dos arquivos pequenos" e por que ele prejudica

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher um formato de tabela (Iceberg / Delta / Hudi) e justificá-lo contra sua carga de trabalho
- Projetar particionamento e tamanho de arquivo para que consultas façam pruning em vez de full-scan
- Implementar tiering (quente/morno/frio) com regras de ciclo de vida e raciocinar sobre tradeoffs de custo/latência
- Usar operações ACID de tabela: append atômico, evolução de schema e time travel
- Medir custo de consulta (bytes varridos, latência) e ligá-lo às decisões de layout

## Requisitos Funcionais

1. O lake deve armazenar dados em um formato de tabela que forneça commits atômicos e evolução de schema.
2. Os dados devem ser particionados para que uma consulta filtrada faça pruning de partições em vez de varrer tudo.
3. Um processo de compactação deve consolidar arquivos pequenos em arquivos de tamanho alvo sem downtime para leitores.
4. O design deve definir tiering com uma política de ciclo de vida documentada (ex.: dados frios → classe de armazenamento mais barata após N dias).
5. A tabela deve suportar time travel: uma consulta deve conseguir ler um snapshot anterior.
6. O custo de consulta (bytes varridos, latência) deve ser mensurável e reportado antes e depois da otimização.

## Marcos Sugeridos

1. **Marco 1 — Formato de tabela no object storage:** Deposite dados em uma tabela Iceberg/Delta/Hudi com um esquema de partição; rode uma consulta base.
2. **Marco 2 — Otimizar layout:** Adicione compactação e dimensione bem os arquivos; meça a queda em bytes varridos e latência.
3. **Marco 3 — Tiering e time travel:** Adicione tiering por ciclo de vida e demonstre ler um snapshot anterior após uma mudança de schema.

## Esboço de Dados e Interface

```text
[ingestão] ─▶ bucket do object store
   warehouse/db/events/
     metadata/           <- formato de tabela: snapshots, schema, lista de manifests
     data/dt=2026-07-24/ part-0001.parquet (alvo ~128-512MB cada)
         dt=2026-07-23/ ...

consulta: SELECT ... WHERE dt = '2026-07-24' AND region = 'us'
   -> prune de partição (dt) -> prune de arquivo via estatísticas de coluna (region)
   -> varre poucos arquivos, não a tabela

job de compactação: muitos arquivos pequenos -> poucos arquivos dimensionados (reescrita atômica)
time travel:   SELECT ... FOR SYSTEM_VERSION AS OF <snapshotId>

Tiering: quente (últimos 7d, standard) | morno (30d, infrequente) | frio (>90d, archive)
Não funcional: consulta p95 < T, custo de storage/TB < $C, sem downtime de leitor na compactação.
```

## Desafios Extras

- Adicione particionamento oculto/por transformação (ex.: `days(ts)` do Iceberg) e compare o pruning contra colunas de partição manuais.
- Implemente um passo de Z-order ou clustering e meça seu efeito em filtros de múltiplas colunas.
- Adicione uma estimativa de custo de consulta guiada por metadados/estatísticas e valide-a contra os bytes realmente varridos.

## Definição de Pronto

- [ ] Os dados estão em um formato de tabela com commits atômicos e schema evoluível.
- [ ] Uma consulta filtrada comprovadamente faz pruning de partições/arquivos em vez de full-scan.
- [ ] A compactação reduz a contagem de arquivos e melhora a latência sem quebrar leituras concorrentes.
- [ ] O time travel retorna um snapshot anterior correto após uma mudança de schema.
- [ ] Um benchmark antes/depois documenta bytes varridos, latência e custo de armazenamento entre tiers.

## Armadilhas Comuns

- O problema dos arquivos pequenos: streaming ou particionamento excessivo produz milhares de arquivos minúsculos que matam o planejamento da consulta.
- Particionar em uma coluna de alta cardinalidade, criando milhões de partições e metadados lentos.
- Tratar o lake como um filesystem e perder atomicidade — dados semi-escritos lidos como completos.
- Ignorar o crescimento de metadados no formato de tabela; snapshots ilimitados também precisam de expiração.

## Recursos

- [Documentação do Apache Iceberg](https://iceberg.apache.org/docs/latest/) — spec da tabela, particionamento e snapshots.
- [Documentação do Delta Lake](https://docs.delta.io/latest/index.html) — transações ACID e time travel no lake.
- [Documentação do Apache Hudi](https://hudi.apache.org/docs/overview) — tradeoffs de copy-on-write vs merge-on-read.
- [Trino: Object storage e pruning](https://trino.io/docs/current/connector/hive.html) — como um motor de consulta faz pruning de dados do lake.
