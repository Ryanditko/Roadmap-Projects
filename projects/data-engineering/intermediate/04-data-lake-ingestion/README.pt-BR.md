# Sistema de Ingestão em Data Lake

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um sistema de ingestão que aterrissa dados brutos de vários formatos em um data lake e os registra em um catálogo para que possam de fato ser encontrados e consultados. O lake em si é apenas armazenamento de objetos (filesystem local, MinIO ou S3), mas um lake sem organização é um pântano. Seu trabalho é a camada que lhe dá estrutura: um layout em zonas (raw → cleaned), particionamento consistente, metadados extraídos e um catálogo que registra o que chegou, quando e em qual schema. Você ingerirá CSV, JSON e Parquet, os normalizará em um formato colunar e tornará a ingestão idempotente para que reprocessar uma entrega substitua em vez de duplicar. A recompensa é entender por que "só jogar os arquivos no S3" é onde a engenharia de dados começa, não termina.

## Pré-requisitos

- Conforto para ler e escrever arquivos em uma linguagem como Python
- Familiaridade com formatos colunares (Parquet) e de linha (CSV/JSON)
- Entendimento básico de armazenamento de objetos e layouts de chave/prefixo
- Acesso a armazenamento de objetos local (filesystem, MinIO ou um bucket na nuvem)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um layout de lake em zonas (raw vs cleaned) com um esquema de partição consistente
- Ingerir formatos heterogêneos e normalizá-los para um único formato colunar
- Extrair e registrar metadados (schema, contagem de linhas, origem, hora de ingestão) em um catálogo
- Tornar a ingestão idempotente para que reprocessar um arquivo não duplique dados
- Suportar um backfill que reingere um intervalo de entregas históricas

## Requisitos Funcionais

1. O sistema deve ingerir ao menos três formatos (ex.: CSV, JSON, Parquet) através de uma interface comum.
2. Arquivos brutos devem aterrissar inalterados em uma zona raw e depois ser normalizados na zona cleaned como Parquet.
3. Os dados devem ser particionados por uma chave estável (ex.: origem e data de ingestão) com um layout de caminho consistente.
4. Cada dataset ingerido deve ser registrado em um catálogo com schema, contagem de linhas, origem e timestamp.
5. Reingerir a mesma entrega deve ser idempotente — ela substitui a partição, não anexa a ela.
6. O sistema deve suportar backfill de um intervalo de datas de entregas sem passos manuais por arquivo.
7. Uma falha de ingestão deve deixar a partição alvo em seu estado consistente anterior, nunca meio escrita.

## Marcos Sugeridos

1. **Marco 1 — Aterrissar raw:** Ingira um formato em uma zona raw particionada com caminhos corretos.
2. **Marco 2 — Normalizar e catalogar:** Converta para Parquet na zona cleaned e registre metadados no catálogo.
3. **Marco 3 — Idempotência e backfill:** Torne a escrita de partição atômica/idempotente e reingira um intervalo histórico.

## Esboço de Dados e Interface

```text
lake/
  raw/     source=orders/ingest_date=2024-01-01/orders.csv
  cleaned/ source=orders/ingest_date=2024-01-01/part-000.parquet

entrada no catálogo:
  dataset:      "orders"
  path:         "cleaned/source=orders/ingest_date=2024-01-01/"
  format:       "parquet"
  schema:       [ {name, type}, ... ]
  row_count:    12045
  source_uri:   "sftp://.../orders.csv"
  ingested_at:  "2024-01-01T02:15:00Z"

ingest(source, ingest_date):
  escreve em <tmp>/... ; ao ter sucesso troca atomicamente para a partição (idempotente)
Backfill: for d in intervalo_datas: ingest("orders", d)
```

## Desafios Extras

- Adicione inferência de schema mais uma checagem que falha a ingestão quando o schema de um arquivo muda inesperadamente.
- Rastreie linhagem simples: qual arquivo raw produziu qual partição cleaned.
- Adicione um formato de tabela aberto (Delta Lake, Apache Iceberg ou Hudi) e o compare a Parquet puro + catálogo.
- Comprima e compacte muitos arquivos pequenos em menos arquivos de tamanho adequado.

## Definição de Pronto

- [ ] Todos os três formatos ingerem por uma interface e aterrissam como Parquet na zona cleaned.
- [ ] As partições seguem um layout de caminho consistente e consultável, chaveado por origem e data.
- [ ] O catálogo reflete o schema, contagem de linhas, origem e hora de ingestão de cada dataset.
- [ ] Reingerir a mesma entrega mantém as contagens de linhas inalteradas (troca de partição idempotente).
- [ ] Um backfill de várias datas popula cada partição e sua entrada no catálogo.

## Armadilhas Comuns

- Escrever diretamente no caminho final da partição, fazendo um crash deixar uma partição meio escrita e ilegível.
- Anexar a uma partição na reingestão em vez de substituí-la, duplicando linhas silenciosamente.
- Deixar cada origem escolher sua própria convenção de caminho, tornando o lake inconsultável como um todo.
- Produzir milhares de arquivos minúsculos, o que prejudica os motores de consulta downstream.
- Guardar metadados só nos nomes dos arquivos em vez de um catálogo real, exigindo listar tudo para descobrir.

## Recursos

- [AWS: O que é um data lake?](https://aws.amazon.com/what-is/data-lake/) — zonas, catálogos e arquitetura de lake.
- [Documentação do Apache Parquet](https://parquet.apache.org/docs/) — o formato colunar e por que ele serve a lakes.
- [Databricks: Arquitetura medallion](https://www.databricks.com/glossary/medallion-architecture) — zoneamento raw/bronze → silver → gold.
- [Documentação do Apache Iceberg](https://iceberg.apache.org/docs/latest/) — um formato de tabela aberto para lakes.
