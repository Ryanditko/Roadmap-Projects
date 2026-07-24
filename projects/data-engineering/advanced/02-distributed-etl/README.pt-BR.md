# Sistema de ETL Distribuído

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete um job de ETL distribuído que lê um dataset grande, o transforma em muitos nós de trabalho e grava um resultado particionado — mantendo-se correto e barato quando um worker morre no meio da execução. Os problemas interessantes aqui não são as transformações em si, mas tudo ao redor: como os dados são particionados, por que uma única chave quente pode travar um estágio inteiro (data skew), como um shuffle move gigabytes pela rede e como o checkpointing permite recuperar sem refazer tudo. Você tratará o cluster como uma máquina não confiável — nós somem, discos enchem, uma partição é 100× as outras — e construirá um job cujo tempo de execução e custo permaneçam previsíveis mesmo assim. A entrega é um design e um job distribuído funcional em um framework como o Spark, mais um plano documentado para skew, retries e recuperação.

## Pré-requisitos

- Conhecimento prático de um framework distribuído (Apache Spark ou Hadoop MapReduce)
- Entendimento de particionamento, shuffles e o modelo mental map/reduce
- Conforto para raciocinar sobre I/O de rede e disco como o custo dominante
- Familiaridade com um formato colunar (Parquet/ORC) e armazenamento de objetos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Particionar a entrada e controlar o paralelismo para casar com os recursos do cluster
- Diagnosticar e mitigar data skew (salting, broadcast joins, reparticionamento)
- Explicar o que um shuffle faz e como minimizar seu custo
- Usar checkpointing e escritas idempotentes para que uma execução falha retome com segurança
- Instrumentar um job distribuído e ler suas métricas de estágio/task para achar gargalos

## Requisitos Funcionais

1. O job deve processar um dataset muito maior que a memória de qualquer nó único, com paralelismo limitado.
2. As transformações devem ser determinísticas e idempotentes para que uma task repetida produza saída idêntica.
3. O job deve detectar e mitigar skew em ao menos uma chave de join ou group-by.
4. Em caso de falha de worker, apenas as tasks perdidas devem ser reexecutadas — não o job inteiro.
5. A saída deve ser gravada particionada (ex.: por data) no armazenamento de objetos, com commit atômico para que escritas parciais nunca sejam lidas como completas.
6. O job deve emitir métricas de registros de entrada/saída, bytes de shuffle e duração por estágio.

## Marcos Sugeridos

1. **Marco 1 — Job base:** Ler → transformar → gravar saída particionada em um cluster pequeno; confirme a correção em uma amostra conhecida.
2. **Marco 2 — Escala e skew:** Rode contra um dataset grande e enviesado; meça o straggler, então aplique salting ou um broadcast join e meça de novo.
3. **Marco 3 — Resiliência:** Adicione checkpointing e escritas atômicas/idempotentes; mate um worker no meio e verifique a recuperação limpa.

## Esboço de Dados e Interface

```text
[origem: object store]  raw/events/*.parquet   (bilhões de linhas)
        │  ler, partições = f(tamanho da entrada, cores)
        ▼
[estágio map]  parse, filtro, colunas derivadas   (sem shuffle)
        │
        ▼
[shuffle]  reparticiona por joinKey  ── skew? salgar chaves quentes: key -> key#rand(0..N)
        │
        ▼
[estágio reduce]  join / agregação                (checkpoint aqui)
        │
        ▼
[sink]  grava particionado por dt=YYYY-MM-DD, commit atômico (marcador _SUCCESS)
        out/agg/dt=2026-07-24/part-*.parquet

Alvos não funcionais: runtime < T para tamanho S, custo < $C,
recuperação reexecuta apenas tasks falhas.
```

## Desafios Extras

- Adicione execução adaptativa de consultas (ou equivalente manual) que reparticiona com base nos tamanhos de shuffle observados.
- Suporte execuções incrementais que processam apenas partições novas em vez do dataset completo.
- Adicione um pool de instâncias spot/preemptíveis e prove que o job ainda conclui quando nós são recuperados.

## Definição de Pronto

- [ ] O job conclui em um dataset maior que a RAM de qualquer nó sem OOM.
- [ ] Uma mitigação de skew documentada encolhe mensuravelmente a task mais lenta.
- [ ] Matar um worker no meio reexecuta apenas as tasks perdidas e gera saída idêntica.
- [ ] A saída é particionada e só fica visível após o commit atômico; uma escrita interrompida não deixa dados parciais legíveis.
- [ ] Um benchmark registra runtime, bytes de shuffle e custo no tamanho de dataset alvo.

## Armadilhas Comuns

- Deixar o framework escolher uma contagem de partições padrão baixa ou alta demais para seus dados.
- Corrigir skew adicionando memória em vez de rebalancear chaves — isso adia a falha, não a remove.
- Escritas não idempotentes, fazendo uma task repetida anexar duplicatas.
- Ler a saída antes de o marcador de commit atômico existir e tratar uma escrita parcial como completa.

## Recursos

- [Spark: Tuning e Performance](https://spark.apache.org/docs/latest/tuning.html) — orientação de memória, serialização e particionamento.
- [Tuning de Performance do Spark SQL](https://spark.apache.org/docs/latest/sql-performance-tuning.html) — broadcast joins e execução adaptativa para skew.
- [MapReduce (artigo)](https://research.google/pubs/pub62/) — o modelo original de processamento distribuído tolerante a falhas.
- [Documentação do Apache Parquet](https://parquet.apache.org/docs/) — o formato colunar e sua história de particionamento.
