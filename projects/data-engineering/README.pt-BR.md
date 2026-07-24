![Data Engineering Projects Banner](../images/banner-data-engineering.png)

> 🌐 [English](./README.md) · **Português**

# Projetos de Engenharia de Dados

Construa a infraestrutura que move, transforma e armazena dados de forma
confiável em escala: pipelines ETL/ELT, streaming, data warehouses e data
lakes. São exercícios guiados — você recebe o *o quê* e o *porquê*, e então
escreve o código.

> Cada projeto tem um brief em inglês (`README.md`) e outro em português
> (`README.pt-BR.md`).

## O Que Você Vai Construir

- Pipelines de ETL em batch e sistemas de ingestão de arquivos
- Workflows orquestrados com schedulers como o Airflow
- Pipelines de streaming com Kafka e processamento em tempo real
- Data warehouses, data lakes e pipelines de CDC
- Infraestrutura distribuída, otimizada em custo e auto-recuperável

## O Que Você Vai Aprender

- **ETL/ELT**: padrões de extração, transformação e carga
- **Orquestração**: DAGs, agendamento, dependências
- **Streaming**: pub/sub, janelas, semântica exactly-once
- **Modelagem**: modelagem dimensional, evolução de schema
- **Confiabilidade**: idempotência, retentativas, monitoramento, qualidade de dados

## Projetos Iniciantes (10 Projetos)

Conceitos fundamentais de engenharia de dados por meio de tarefas focadas.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [CSV to Database Loader](./beginner/01-csv-to-database/) | Carregue dados CSV em um banco de dados SQL |
| 2 | [Simple ETL Pipeline](./beginner/02-simple-etl/) | Extraia, transforme e carregue dados de uma fonte para outra |
| 3 | [Data Validation Script](./beginner/03-data-validation/) | Valide a qualidade dos dados com verificações e regras |
| 4 | [Log Parser](./beginner/04-log-parser/) | Analise e estruture arquivos de log para análise |
| 5 | [API to CSV Exporter](./beginner/05-api-to-csv/) | Busque dados de API e exporte para CSV |
| 6 | [Batch Processing Script](./beginner/06-batch-processing/) | Processe grandes datasets em lotes |
| 7 | [File Ingestion System](./beginner/07-file-ingestion/) | Construa um sistema para ingerir múltiplos tipos de arquivo |
| 8 | [Data Cleaning Pipeline](./beginner/08-data-cleaning/) | Crie workflows reutilizáveis de limpeza de dados |
| 9 | [JSON Transformer](./beginner/09-json-transformer/) | Transforme dados JSON em diferentes formatos |
| 10 | [Basic Scheduler](./beginner/10-basic-scheduler/) | Agende e execute tarefas de dados automaticamente |

## Projetos Intermediários (10 Projetos)

Vários conceitos combinados em padrões reais de engenharia de dados.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [ETL Pipeline with Airflow](./intermediate/01-airflow-etl/) | Construa pipelines orquestrados com Apache Airflow |
| 2 | [Data Warehouse Loader](./intermediate/02-data-warehouse-loader/) | Carregue dados em um sistema de data warehouse |
| 3 | [Streaming Pipeline with Kafka](./intermediate/03-streaming-kafka/) | Processe streams de dados em tempo real com Kafka |
| 4 | [Data Lake Ingestion System](./intermediate/04-data-lake-ingestion/) | Construa infraestrutura de data lake para dados brutos |
| 5 | [Schema Evolution System](./intermediate/05-schema-evolution/) | Lide com schemas de dados que mudam nos pipelines |
| 6 | [Incremental Data Processing](./intermediate/06-incremental-processing/) | Processe apenas dados novos/alterados de forma eficiente |
| 7 | [Pipeline Monitoring System](./intermediate/07-pipeline-monitoring/) | Monitore e alerte sobre a saúde dos pipelines |
| 8 | [Data Quality Framework](./intermediate/08-data-quality/) | Construa um framework abrangente de validação de dados |
| 9 | [Multi-Source Ingestion](./intermediate/09-multi-source-ingestion/) | Combine dados de múltiplas fontes |
| 10 | [Partitioned Data Pipeline](./intermediate/10-partitioned-pipeline/) | Construa processamento particionado eficiente |

## Projetos Avançados (10 Projetos)

Arquitete infraestrutura de dados complexa com preocupações de nível empresarial.

| # | Projeto | Descrição |
|---|---------|-----------|
| 1 | [Real-Time Data Platform](./advanced/01-real-time-platform/) | Construa infraestrutura de dados em tempo real de ponta a ponta |
| 2 | [Distributed ETL System](./advanced/02-distributed-etl/) | Escale ETL entre múltiplas máquinas e clusters |
| 3 | [Data Mesh Simulation](./advanced/03-data-mesh/) | Implemente arquitetura de dados descentralizada |
| 4 | [Scalable Data Lake Architecture](./advanced/04-data-lake-architecture/) | Projete um data lake para escala massiva |
| 5 | [CDC Pipeline](./advanced/05-cdc-pipeline/) | Implemente captura de mudanças (CDC) dos sistemas de origem |
| 6 | [Data Lineage System](./advanced/06-data-lineage/) | Rastreie o fluxo de dados e dependências entre pipelines |
| 7 | [High-Throughput Streaming](./advanced/07-high-throughput-streaming/) | Processe milhões de eventos por segundo |
| 8 | [Cost-Optimized Pipeline](./advanced/08-cost-optimized-pipeline/) | Construa pipelines eficientes minimizando custos de nuvem |
| 9 | [Multi-Region Replication](./advanced/09-multi-region-replication/) | Replique dados entre regiões geográficas |
| 10 | [Self-Healing Pipelines](./advanced/10-self-healing-pipelines/) | Construa mecanismos de recuperação automatizados |

## Trilha de Aprendizado

- **Iniciante (3–4 semanas)**: fundamentos de SQL, conceitos de ETL, pipelines
  e loaders simples, manipulação de CSV/JSON.
- **Intermediário (6–8 semanas)**: orquestração com Airflow, data warehousing,
  streaming com Kafka, monitoramento e validação.
- **Avançado (2–3 meses)**: sistemas distribuídos, arquiteturas escaláveis,
  data mesh e CDC, otimização de custo e desempenho.

Projete para confiabilidade e observabilidade desde o primeiro pipeline.

## Conceitos-Chave

1. **SQL** — consultas, índices, transações
2. **Design de ETL** — padrões de extração, transformação e carga
3. **Modelagem de dados** — schema e modelagem dimensional
4. **Escalabilidade** — particionamento, sharding, distribuição
5. **Confiabilidade** — tratamento de erros, retentativas, idempotência
6. **Monitoramento** — alertas, logging, métricas
7. **Qualidade de dados** — validação, reconciliação, governança

## Recursos

- [Tutorial de SQL do PostgreSQL](https://www.postgresql.org/docs/current/tutorial.html)
- [Documentação do Apache Airflow](https://airflow.apache.org/)
- [Documentação do dbt](https://docs.getdbt.com/)
- [Guia do Apache Spark](https://spark.apache.org/docs/latest/)
- [Roadmap de Engenharia de Dados](https://roadmap.sh/data-engineer)

## Próximos Passos

1. Fique confortável com SQL e um banco de dados relacional.
2. Escolha um projeto iniciante e leia o brief até o fim.
3. Comece com volumes pequenos de dados e escale gradualmente.
4. Adicione monitoramento e validação e depois tente os desafios extras.

**Escolha um projeto e comece a construir infraestrutura de dados.**
