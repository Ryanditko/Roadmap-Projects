# Script de Processamento em Lote

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Processar um milhão de registros um a um é lento e frágil; processá-los todos de uma vez estoura sua memória. A resposta é o lote (batch): divida a entrada em grupos de tamanho fixo, processe cada grupo como uma unidade e faça o commit antes de seguir. Construa um script que lê uma entrada grande, a percorre em lotes, trata a falha de um único lote sem perder os já concluídos, e consegue retomar de onde parou. Este é o padrão cavalo de batalha da engenharia de dados — aquele ao qual você recorre sempre que "só percorra tudo" para de funcionar.

## Pré-requisitos

- Conforto para ler um arquivo de entrada grande ou resultado de consulta iterativamente
- Entender transações (um lote ou faz commit completo ou rollback)
- Tratamento básico de erros (try/except ou equivalente)
- Familiaridade com ler e escrever um pequeno arquivo de estado/checkpoint

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Dividir um fluxo de registros em lotes de tamanho fixo sem carregar tudo
- Processar cada lote como uma unidade atômica que faz commit ou rollback de forma limpa
- Isolar um lote com falha para que lotes bem-sucedidos anteriores não sejam perdidos
- Registrar checkpoints do progresso para que uma execução interrompida retome, e não recomece
- Reportar progresso por lote e geral com contagens e tempo
- Raciocinar sobre o trade-off entre tamanho do lote, memória e throughput

## Requisitos Funcionais

1. O script deve ler a entrada e agrupar registros em lotes de um tamanho configurável.
2. Cada lote deve ser processado transacionalmente: em caso de erro, esse lote faz rollback mas a execução continua.
3. Um lote com falha deve ser registrado (seu identificador e motivo) para inspeção ou retry posterior.
4. O script deve escrever um checkpoint após cada lote bem-sucedido para poder retomar do último ponto confirmado.
5. Ao reiniciar, deve pular os lotes já processados e continuar a partir do checkpoint.
6. Deve reportar progresso: lotes concluídos, registros processados, falhas e tempo total.

## Marcos Sugeridos

1. **Marco 1 — Lote e processamento:** Divida a entrada e processe cada lote, imprimindo o progresso.
2. **Marco 2 — Isolamento de falhas:** Envolva cada lote em uma transação e registre falhas sem abortar.
3. **Marco 3 — Retomada:** Adicione checkpoints e lógica de pular-concluídos para que uma execução interrompida continue.

## Esboço de Dados e Interface

```text
entrada: 1.000.000 registros (stream)
tamanho do lote: 500  -> 2.000 lotes

fluxo por lote
  acumula 500 registros
  inicia transação
    processa + grava o lote
  commit         -> checkpoint {last_batch: N, records: N*500}
  em erro        -> rollback, log {batch: N, reason}, continua

arquivo de checkpoint (checkpoint.json)
  { "last_completed_batch": 1450, "records_done": 725000 }

reinício
  lê o checkpoint -> retoma no lote 1451

CLI: process --input data.jsonl --batch-size 500 --resume
relatório: lotes=2000 ok=1996 falhos=4 registros=998000 tempo=42s
```

## Desafios Extras

- Adicionar um modo `--retry-failed` que reprocessa apenas os lotes registrados como falhos.
- Processar lotes independentes em paralelo com um pool limitado de workers.
- Tornar o tamanho do lote adaptativo, encolhendo após uma falha e crescendo quando estável.
- Emitir uma linha de métricas por lote (throughput em registros/seg) para tuning.

## Definição de Pronto

- [ ] A entrada é processada em lotes de tamanho configurável sem carregá-la toda.
- [ ] Um único lote com falha é isolado e registrado; os outros lotes ainda concluem.
- [ ] Um checkpoint é escrito após cada lote bem-sucedido.
- [ ] Matar e reiniciar o script retoma a partir do checkpoint, não do início.
- [ ] O relatório final mostra lotes, registros, falhas e tempo decorrido.

## Armadilhas Comuns

- Fazer commit por registro (lento demais) ou por execução inteira (perde tudo em uma falha).
- Deixar um lote ruim lançar uma exceção não tratada e matar o job inteiro.
- Fazer checkpoint antes do lote de fato dar commit, de modo que um crash perde trabalho confirmado ou processa em dobro.
- Escolher um tamanho de lote às cegas em vez de medir memória e throughput.

## Recursos

- [PostgreSQL: Transações](https://www.postgresql.org/docs/current/tutorial-transactions.html) — a garantia de atomicidade da qual um lote depende.
- [Receitas do `itertools` do Python](https://docs.python.org/3/library/itertools.html#itertools-recipes) — um `batched()` limpo para dividir iteráveis.
- [Idempotência e checkpointing](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) — por que retomadas seguras precisam de idempotência.
- [Designing Data-Intensive Applications (conceitos)](https://dataintensive.net/) — fundamentos de processamento em lote no capítulo 10.
