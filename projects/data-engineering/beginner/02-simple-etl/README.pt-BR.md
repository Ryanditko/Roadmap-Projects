# Pipeline ETL Simples

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um pequeno pipeline de Extract-Transform-Load (Extrair-Transformar-Carregar) que lê registros de um lugar, os remodela e os grava em outro. A versão clássica para iniciantes: puxe linhas de um arquivo ou tabela de origem, limpe e enriqueça-as, e as deposite em uma tabela de destino. O valor aqui não está em nenhuma etapa isolada, mas no *formato* do todo — três estágios claramente separados unidos por um formato de registro bem definido. Quando você conseguir enxergar extração, transformação e carga como funções independentes e testáveis, você terá a espinha dorsal sobre a qual todo pipeline de dados, por maior que seja, é construído.

## Pré-requisitos

- Conforto para ler e escrever arquivos ou um banco de dados simples (veja [Carregador de CSV para Banco de Dados](../01-csv-to-database/) primeiro, se isso for novo)
- Funções e estruturas de dados básicas na linguagem de sua escolha
- Entender como um registro/linha se parece como um dicionário ou objeto
- Familiaridade com a execução de um script pela linha de comando

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Separar um pipeline em estágios distintos de extração, transformação e carga
- Modelar um registro em trânsito como uma estrutura de dados simples passada entre os estágios
- Aplicar transformações no nível de campo: renomear, derivar e converter tipos
- Rotear registros ruins para um destino de rejeições em vez de travar a execução
- Emitir metadados da execução (contagens, duração) para que o pipeline seja observável
- Projetar o estágio de transformação para ser puro e testável por testes unitários

## Requisitos Funcionais

1. O pipeline deve ter três estágios separáveis, cada um chamável de forma independente para testes.
2. A extração deve ler de uma origem definida e produzir registros um a um.
3. A transformação deve aplicar ao menos três operações: uma renomeação, um campo derivado e uma conversão de tipo.
4. Um registro que falhar na transformação deve ser enviado a um destino de rejeições com o motivo, sem parar o pipeline.
5. A carga deve gravar registros válidos no destino e ser segura para re-execução.
6. O pipeline deve imprimir um resumo: registros extraídos, carregados e rejeitados, além do tempo decorrido.

## Marcos Sugeridos

1. **Marco 1 — Extrair e repassar:** Leia a origem e carregue registros inalterados no destino.
2. **Marco 2 — Transformar:** Insira o estágio de transformação com renomeações, campos derivados e conversões.
3. **Marco 3 — Resiliência e relatório:** Adicione o destino de rejeições e um resumo da execução com contagens e tempo.

## Esboço de Dados e Interface

```text
extração (origem: sales.csv)
  {"order":"A1","amount":"19.90","ts":"2024-03-01T10:00Z"}

transformação
  order  -> order_id     (renomeia)
  amount -> amount_cents (float * 100 -> int; rejeita se não numérico)
  ts     -> order_date   (deriva a data do timestamp)

carga (destino: tabela orders)
  order_id | amount_cents | order_date

destino de rejeições -> rejects.jsonl
  {"record": {...}, "reason": "amount não numérico"}

fluxo: extração -> transformação -> [ok]  carga
                                 \-> [ruim] rejeições
resumo: extraídos=500 carregados=492 rejeitados=8 tempo=1.4s
```

## Desafios Extras

- Adicionar extração incremental usando uma marca d'água (só linhas novas desde a última execução).
- Tornar a transformação orientada por configuração para que os mapeamentos morem em um arquivo, não no código.
- Suportar múltiplos destinos (uma tabela mais um arquivo Parquet) em uma única execução.
- Adicionar uma flag `--limit` para processar uma amostra para iteração rápida.

## Definição de Pronto

- [ ] Cada estágio pode ser chamado e testado isoladamente.
- [ ] Uma origem válida roda de ponta a ponta e deposita registros corretos.
- [ ] Registros ruins caem no destino de rejeições com um motivo e não param a execução.
- [ ] Re-executar o pipeline não duplica registros carregados.
- [ ] O resumo reporta contagens de extraídos, carregados e rejeitados, além do tempo.

## Armadilhas Comuns

- Fundir os três estágios em uma única função, tornando tudo não testável.
- Deixar um registro ruim lançar exceção e matar o lote inteiro.
- Mutar registros de origem no lugar em vez de produzir novos transformados.
- Esquecer da normalização de fuso horário ou de formato ao derivar datas.

## Recursos

- [Python Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html) — geradores e funções puras para estágios de pipeline.
- [Wikipedia: Extract, transform, load](https://en.wikipedia.org/wiki/Extract,_transform,_load) — o padrão e seu vocabulário.
- [Conceitos do Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html) — como a indústria orquestra ETL real, vale ler para contexto.
- [Documentação do petl](https://petl.readthedocs.io/en/stable/) — uma biblioteca leve de ETL em Python para comparar com sua versão feita à mão.
