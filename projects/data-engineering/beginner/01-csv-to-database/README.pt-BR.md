# Carregador de CSV para Banco de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Pegue um arquivo CSV bruto — do tipo exportado de uma planilha ou de um sistema legado — e carregue-o de forma limpa em uma tabela de banco de dados relacional. No caminho você vai encarar as perguntas que todo job de ingestão acaba fazendo: qual é o tipo de cada coluna, o que faço com a linha que tem uma letra onde deveria haver um número, e como re-executo isso amanhã sem duplicar todos os registros? Este projeto mantém o escopo pequeno (um arquivo, uma tabela) para você focar em fazer a carga *corretamente* em vez de *rápido*, e te dá um modelo mental reutilizável para o padrão CSV-para-warehouse que aparece em toda parte no trabalho com dados.

## Pré-requisitos

- Python básico (ou uma linguagem com biblioteca de CSV e driver de banco)
- Um banco relacional que você consiga rodar localmente (SQLite não exige configuração; Postgres é um bom próximo passo)
- Conforto para escrever comandos simples de `CREATE TABLE` e `INSERT`
- Entender o que é uma chave primária e por que ela importa

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Ler um arquivo CSV linha a linha em vez de carregá-lo todo na memória
- Inferir ou declarar o tipo de uma coluna e converter valores de texto para ele
- Projetar um esquema de tabela que corresponda aos seus dados de origem
- Realizar inserções em lotes dentro de uma transação
- Pular ou colocar em quarentena linhas malformadas sem abortar toda a carga
- Tornar o carregador idempotente para que uma re-execução não crie duplicatas

## Requisitos Funcionais

1. A ferramenta deve ler um CSV com linha de cabeçalho e mapear cada coluna para um campo da tabela por nome, não por posição.
2. A ferramenta deve criar a tabela de destino caso ela ainda não exista.
3. Cada valor deve ser convertido para o tipo declarado; uma linha que falhar na conversão deve ser rejeitada, registrada com seu número de linha e pulada — nunca descartada silenciosamente.
4. As linhas devem ser inseridas dentro de uma transação para que uma falha no meio da carga não deixe um lote parcial.
5. Re-executar o carregador no mesmo arquivo não deve criar linhas duplicadas (use uma chave natural ou primária).
6. Ao concluir, a ferramenta deve reportar contagens: linhas lidas, inseridas e rejeitadas.

## Marcos Sugeridos

1. **Marco 1 — Analisar e imprimir:** Leia o CSV e imprima as linhas tipadas no console; ainda sem banco de dados.
2. **Marco 2 — Carregar:** Crie a tabela e insira as linhas em transações em lote.
3. **Marco 3 — Robustez:** Adicione conversão de tipos com um log de rejeições, re-execuções idempotentes e um relatório de resumo.

## Esboço de Dados e Interface

```text
origem: users.csv
  id,name,signup_date,score
  1,Ana,2024-01-05,88

transformação:
  id          -> INTEGER   (rejeita se não for parseável)
  name        -> TEXT      (remove espaços)
  signup_date -> DATE      (parse ISO-8601)
  score       -> REAL      (NULL se vazio)

tabela de destino: users (id PRIMARY KEY, name, signup_date, score)

rejects.log: linha 42: score="N/A" não é número -> pulada

CLI: loader --file users.csv --table users --batch 500
resumo: lidas=1000 inseridas=987 rejeitadas=13
```

## Desafios Extras

- Inferir os tipos das colunas automaticamente amostrando as primeiras N linhas.
- Suportar "upsert" para que chaves existentes sejam atualizadas em vez de puladas.
- Adicionar uma flag `--dry-run` que valida sem escrever.
- Ler um CSV compactado com gzip sem descomprimi-lo totalmente em disco.

## Definição de Pronto

- [ ] Um CSV bem-formado carrega totalmente na tabela com os tipos corretos.
- [ ] Linhas malformadas são registradas com números de linha e puladas.
- [ ] Rodar o carregador duas vezes resulta na mesma contagem de linhas que uma vez.
- [ ] Uma falha no meio da carga faz rollback da transação atual de forma limpa.
- [ ] O resumo final reporta contagens de lidas, inseridas e rejeitadas.

## Armadilhas Comuns

- Ler o arquivo inteiro na memória — use um leitor em streaming para que arquivos grandes não esgotem a RAM.
- Inserir uma linha por comando, o que é lento; agrupe em lotes dentro de uma transação.
- Tratar strings vazias como números ou datas válidos; decida o tratamento de NULL explicitamente.
- Assumir que as posições das colunas nunca mudam em vez de mapear pelo nome do cabeçalho.

## Recursos

- [Módulo `csv` do Python](https://docs.python.org/3/library/csv.html) — o leitor de CSV em streaming padrão.
- [Documentação do SQLite](https://www.sqlite.org/docs.html) — banco sem configuração, ideal para este projeto.
- [PostgreSQL `COPY`](https://www.postgresql.org/docs/current/sql-copy.html) — o caminho rápido de carga em massa quando você superar as inserções linha a linha.
- [pandas `read_csv`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html) — uma opção de mais alto nível que vale comparar com o parsing manual.
