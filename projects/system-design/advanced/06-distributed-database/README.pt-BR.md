# Projete um Banco de Dados Distribuído

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete um banco de dados distribuído que forneça transações ACID entre muitos nós e datacenters. Diferente de projetar uma aplicação sobre um banco de dados, aqui o banco *é* o sistema: você decide como os dados são particionados, como as réplicas concordam sobre escritas via consenso, como as transações fazem commit atomicamente entre shards e o que acontece quando a rede divide um cluster em dois. É aqui que o CAP deixa de ser um slogan e vira um conjunto concreto de escolhas de engenharia. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Domínio firme de ACID, níveis de isolamento e os teoremas CAP/PACELC
- Entendimento de algoritmos de consenso (Raft, Paxos) ao menos conceitualmente
- Familiaridade com replicação (líder/seguidor, quórum) e particionamento
- Exposição a protocolos de transação distribuída (two-phase commit)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher um esquema de particionamento (range vs hash) e justificá-lo contra a carga de trabalho
- Projetar replicação com um grupo de consenso por shard e raciocinar sobre tamanhos de quórum
- Especificar um protocolo de transação entre shards e sua garantia de isolamento
- Analisar o comportamento sob partição de rede e posicionar seu sistema no espectro CAP
- Planejar resharding, recuperação de falhas e replicação multi-datacenter

## Requisitos e Restrições

1. Fornecer transações serializáveis ou de snapshot isolation entre shards.
2. Sobreviver à perda de uma minoria de réplicas em qualquer shard sem perda de dados.
3. Particionar dados para escalar escritas horizontalmente; suportar resharding online.
4. Limitar a latência de commit; documentar o custo de latência de commits cross-shard/cross-região.
5. Declarar a postura CAP explicitamente: sob partição, favorecer consistência (rejeitar escritas) — CP.
6. Suportar replicação multi-datacenter com um trade-off de consistência/latência definido.
7. Fornecer backup, recuperação point-in-time e evolução de schema sem downtime.

## Abordagem Sugerida

Comece fixando o alvo de consistência — digamos, serializable snapshot isolation — porque ele restringe todo o resto. Particione o keyspace (particionamento por range permite scans eficientes; hash distribui a carga uniformemente e evita hotspots). Replique cada partição com um grupo de consenso (Raft) para que um quórum majoritário faça commit de escritas e tolere uma falha minoritária. Para transações que abrangem shards, sobreponha two-phase commit ao consenso por shard (o modelo Spanner) e use um mecanismo de ordenação por tempo ou timestamp para ordenação global. Analise o caso de partição: como CP, o lado minoritário rejeita escritas para preservar consistência. Projete o resharding como um split/merge em background que move ranges sem parar o mundo.

## Esboço de Arquitetura

```text
Cliente ──> Coordenador (por txn) ──> rotear por chave ──> Grupos de shard
                                                            cada shard = grupo Raft:
                                                              líder + seguidores (commit por quórum)

Txn cross-shard:
  Coordenador: BEGIN -> adquire locks nos shards participantes
             -> PREPARE (cada shard: vota duravelmente via seu grupo Raft)
             -> todos SIM? COMMIT : ABORT  (2PC sobre consenso)
  ordenação por timestamp dá serializabilidade global

Particionamento: keyspace -> [range ou hash] -> shards -> resharded por split/merge

APIs principais (tipo SQL ou KV):
BEGIN / COMMIT / ROLLBACK
PUT(key, value) / GET(key) @ snapshot_ts
scan(range) @ snapshot_ts

Replicação (por shard):
Raft{ term, log[], commitIndex }  # quórum majoritário, minoria pode atrasar/falhar
```

## Tópicos de Aprofundamento

- **CAP/PACELC na prática:** por que este design é CP; o custo de disponibilidade sob partição.
- **Consenso:** eleição de líder no Raft, replicação de log, matemática de quórum, mudanças de membros.
- **Níveis de isolamento:** serializável vs snapshot isolation; como o MVCC fornece leituras.
- **Commit cross-shard:** two-phase commit sobre consenso; a ideia do TrueTime do Spanner.
- **Resharding e recuperação:** splits de range online, catch-up de réplica, backup/PITR.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com as camadas de partição/replicação/transação, refinado.
- [ ] Uma declaração CAP/PACELC explícita com a análise de comportamento sob partição.
- [ ] A estratégia escolhida de particionamento e replicação com justificativa de tamanho de quórum.
- [ ] O protocolo de transação cross-shard e sua garantia de isolamento, detalhados.
- [ ] Uma análise de falha/DR: perda de líder, partição minoritária, perda de datacenter inteiro, resharding no meio de uma escrita.

## Armadilhas Comuns

- Alegar "CA" sob CAP — uma partição vai acontecer; você deve escolher C ou A, não ambos.
- Usar two-phase commit puro sem consenso, fazendo um crash de coordenador bloquear para sempre.
- Particionamento por range em uma chave monotonicamente crescente, criando um hotspot de escrita permanente.
- Ignorar desvio de relógio na ordenação por timestamp, quebrando a serializabilidade global.
- Resharding que para o mundo em vez de mover ranges online.

## Recursos

- [Spanner: o Banco de Dados Globalmente Distribuído do Google](https://research.google/pubs/pub39966/) — TrueTime e transações distribuídas externamente consistentes.
- [In Search of an Understandable Consensus Algorithm (Raft)](https://raft.github.io/raft.pdf) — o artigo do Raft.
- [Dynamo: o Key-value Store Altamente Disponível da Amazon](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — o contraponto AP ao CP do Spanner.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — replicação, particionamento e transações num só lugar.
- [Jepsen: análises de consistência](https://jepsen.io/analyses) — como bancos de dados reais se comportam sob partição.
