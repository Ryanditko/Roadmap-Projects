# Processamento Incremental de Dados

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um job em lote que processa apenas os registros novos ou alterados desde sua última execução bem-sucedida, em vez de reprocessar toda a fonte a cada vez. Esse é o padrão que sustenta as cargas noturnas de data warehouse e os pipelines de CDC: você rastreia uma marca d'água (um timestamp ou id monotônico), lê o delta acima dela e avança a marca somente depois que a escrita é confirmada. A parte interessante não é o SQL — é a contabilidade. O que acontece quando o job falha no meio da escrita, quando uma linha chega atrasada com um timestamp antigo, ou quando você precisa reprocessar ontem? Acertar esses casos é a diferença entre um pipeline em que você confia e um que você precisa vigiar.

## Pré-requisitos

- Conforto para escrever transformações em lote sobre uma fonte tabular (uma tabela de warehouse, arquivos ou uma API)
- Entendimento de timestamps, marcas d'água e identificadores monotônicos
- Familiaridade com idempotência — executar a mesma operação duas vezes gera o mesmo resultado
- Um motor à sua escolha (SQL + um agendador, Spark, DuckDB ou um script simples)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Detectar linhas novas e alteradas usando uma marca d'água em vez de uma varredura completa
- Persistir e avançar o estado de processamento com segurança entre execuções
- Tornar as escritas idempotentes para que uma nova tentativa nunca conte em dobro
- Distinguir carga completa de carga incremental e escolher entre elas
- Tratar dados que chegam atrasados e suportar o reprocessamento de uma janela passada

## Requisitos Funcionais

1. O job deve ler apenas registros com uma chave de mudança maior que a última marca d'água confirmada.
2. O job deve persistir a nova marca d'água somente após a escrita correspondente ser bem-sucedida.
3. Reexecutar o job sem novos dados de origem não deve produzir alterações (idempotente).
4. O job deve suportar um modo de carga completa que reconstrói o destino do zero.
5. O job deve aceitar um intervalo explícito de data/id para reprocessar uma janela passada.
6. Os upserts devem usar a chave natural do registro para que uma linha alterada atualize no lugar, sem duplicar.
7. O job deve registrar metadados da execução: linhas lidas, linhas escritas, marca d'água antes/depois e status.

## Marcos Sugeridos

1. **Marco 1 — Carga completa:** Leia toda a fonte e escreva o destino uma vez, capturando a marca d'água inicial.
2. **Marco 2 — Carga delta:** Leia apenas as linhas acima da marca d'água, faça upsert e avance a marca após o commit.
3. **Marco 3 — Recuperação e reprocessamento:** Torne as reexecuções idempotentes, adicione um intervalo de backfill e trate chegadas atrasadas com uma janela de retrospecto.

## Esboço de Dados e Interface

```text
source.orders
  id           bigint    (chave natural)
  updated_at   timestamp (chave de mudança / fonte da marca d'água)
  ...payload

pipeline_state
  pipeline     string    (ex.: "orders_incremental")
  watermark    timestamp
  updated_at   timestamp

fluxo da execução:
  1. leia  W  = state.watermark
  2. rows    = source onde updated_at > W - lookback   (lookback captura dados atrasados)
  3. upsert rows no destino por id
  4. W' = max(updated_at) entre as linhas
  5. faça commit do destino, então defina state.watermark = W'

modos: incremental (padrão) | full-refresh | backfill(de, até)
```

## Desafios Extras

- Adicione tratamento de exclusões físicas consumindo um change stream ou comparando com um snapshot.
- Rastreie a linhagem por execução para responder "qual execução produziu esta linha?".
- Detecte e alerte quando a marca d'água parar de avançar (um pipeline travado).
- Suporte workers paralelos particionando o intervalo do delta e confirmando o estado de forma atômica.

## Definição de Pronto

- [ ] Uma segunda execução sem novos dados não escreve nada e mantém a marca d'água inalterada.
- [ ] Matar o job após a escrita, mas antes do commit do estado, e reexecutar não produz duplicatas.
- [ ] Uma linha de origem alterada atualiza a linha existente no destino em vez de inserir uma nova.
- [ ] Reprocessar um intervalo passado reprocessa exatamente aquela janela sem perturbar dados mais novos.
- [ ] Os metadados da execução são persistidos e legíveis para cada execução.

## Armadilhas Comuns

- Avançar a marca d'água antes de o commit da escrita — uma falha então pula linhas silenciosamente para sempre.
- Usar `>=` na marca d'água e reprocessar a linha da borda, ou `>` e perdê-la — escolha uma e mantenha a consistência.
- Assumir que `updated_at` é estritamente crescente; desvio de relógio e escritas atrasadas exigem uma folga de lookback.
- Anexar em vez de fazer upsert, fazendo linhas alteradas se acumularem como duplicatas.
- Fazer a carga completa truncar o destino antes de a nova carga ter sucesso, deixando uma tabela vazia em caso de falha.

## Recursos

- [Airbyte: Modos de sincronização incremental](https://docs.airbyte.com/using-airbyte/core-concepts/sync-modes/incremental-append) — como uma ferramenta real modela cargas incrementais vs completas.
- [dbt: Modelos incrementais](https://docs.getdbt.com/docs/build/incremental-models) — builds incrementais baseados em marca d'água na prática.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — capítulos sobre captura de mudanças e dados derivados.
- [Wikipedia: Change data capture](https://en.wikipedia.org/wiki/Change_data_capture) — a família mais ampla de técnicas.
