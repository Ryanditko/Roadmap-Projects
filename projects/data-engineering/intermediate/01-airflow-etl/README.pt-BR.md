# Pipeline ETL com Airflow

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um pipeline ETL agendado que extrai de uma fonte (uma API ou um banco operacional), transforma os dados e os carrega em um warehouse ou tabela analítica — tudo orquestrado pelo Apache Airflow. A parte interessante não é a transformação em si, mas a orquestração ao redor dela: expressar dependências como um DAG, agendar execuções diárias e tornar cada execução *idempotente*, para que reprocessar uma data produza o mesmo resultado em vez de duplicar linhas. Você adicionará um sensor que espera os dados de origem chegarem antes de o DAG prosseguir, configurará retries e alertas para que uma falha transitória se cure sozinha, e suportará um backfill que reprocessa um intervalo de datas históricas. Ao final, você entenderá por que times de dados recorrem a um agendador em vez de um cron e uma pilha de scripts shell.

## Pré-requisitos

- Conforto para escrever Python e raciocinar sobre funções com efeitos colaterais
- SQL básico e o entendimento do que é um job em lote
- Familiaridade com conceitos de agendamento estilo cron
- Um ambiente Airflow local (Docker Compose ou `airflow standalone`) — não instale em um cluster compartilhado

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um fluxo de dados como um DAG com dependências explícitas entre tarefas
- Parametrizar tarefas pela data lógica de execução para que as execuções sejam isoladas e reproduzíveis
- Tornar uma carga idempotente para que reprocessar uma data nunca duplique dados
- Usar um sensor para esperar uma dependência upstream antes de prosseguir
- Fazer o backfill de um intervalo de datas históricas e raciocinar sobre retries e alertas

## Requisitos Funcionais

1. O pipeline deve ser definido como um DAG do Airflow com agendamento diário e dependências claras entre tarefas.
2. Cada tarefa deve derivar sua partição de entrada e saída da data lógica (de execução) do run, não de `now()`.
3. A carga de uma dada data deve ser idempotente — reprocessá-la substitui as linhas daquela data em vez de anexar duplicatas.
4. Um sensor ou equivalente deve bloquear o DAG até que os dados de origem daquela data estejam disponíveis.
5. Tarefas que falham devem repetir com backoff um número limitado de vezes, e retries esgotados devem disparar um alerta.
6. O DAG deve suportar um backfill de um intervalo arbitrário de datas sem edição manual por data.
7. O pipeline deve registrar metadados do run (linhas processadas, duração) para inspeção posterior.

## Marcos Sugeridos

1. **Marco 1 — DAG linear:** Extrair → transformar → carregar como três tarefas ordenadas em agendamento diário.
2. **Marco 2 — Carga idempotente e sensor:** Faça a carga sobrescrever por data e a proteja atrás de um sensor de prontidão.
3. **Marco 3 — Confiabilidade e backfill:** Adicione retries com backoff, alertas de falha, métricas de run e faça o backfill de uma semana.

## Esboço de Dados e Interface

```text
DAG: daily_sales_etl   schedule=@daily   start_date=2024-01-01

  wait_for_source (sensor: source/{{ ds }} existe?)
        |
     extract  --> raw/date={{ ds }}/data.parquet
        |
    transform --> tabela de staging (partição = ds)
        |
      load    --> DELETE WHERE dt = {{ ds }}; INSERT ...   (idempotente)
        |
  record_metrics --> runs(dag_id, ds, linhas, segundos)

{{ ds }} = data lógica de execução do Airflow (YYYY-MM-DD), NÃO "hoje"
Config da tarefa: retries=3, retry_delay=5m, sla=2h, on_failure_callback=notify
Backfill: airflow dags backfill -s 2024-01-01 -e 2024-01-07 daily_sales_etl
```

## Desafios Extras

- Adicione uma tarefa de branching que pula a carga quando a extração retorna zero linhas.
- Gere o DAG dinamicamente a partir de um arquivo de config listando múltiplas tabelas de origem.
- Use mapeamento dinâmico de tarefas para paralelizar sobre partições.
- Adicione uma SLA para que um run que perde o prazo dispare um alerta distinto.

## Definição de Pronto

- [ ] O DAG renderiza na UI do Airflow com dependências corretas e sem erros de import.
- [ ] Reprocessar qualquer data produz saída idêntica — verificado por contagens de linhas antes e depois.
- [ ] A carga nunca inicia até o sensor de prontidão confirmar que os dados de origem existem.
- [ ] Um backfill de vários dias completa e popula uma partição correta por data.
- [ ] Uma falha forçada de tarefa repete conforme a config e dispara o alerta configurado.

## Armadilhas Comuns

- Usar `datetime.now()` dentro das tarefas em vez da data de execução, o que quebra backfills e a reprodutibilidade.
- Anexar na carga, fazendo reprocessamentos duplicarem os dados silenciosamente — sempre torne a escrita idempotente.
- Fazer computação pesada no escopo de topo do arquivo do DAG, que roda a cada parse do scheduler.
- Definir `retries` sem `retry_delay`, fazendo uma dependência instável martelar a fonte.
- Tratar retries como substituto de idempotência; um retry de uma tarefa não idempotente corrompe os dados.

## Recursos

- [Apache Airflow: Conceitos Centrais](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/index.html) — DAGs, tarefas, operadores e o scheduler.
- [Airflow: Boas Práticas](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html) — idempotência, código de topo e testes.
- [Astronomer: boas práticas de DAG no Airflow](https://www.astronomer.io/docs/learn/dag-best-practices) — padrões práticos para pipelines confiáveis.
- [O roadmap de Data Engineer](https://roadmap.sh/data-engineer) — onde a orquestração se encaixa no quadro maior.
