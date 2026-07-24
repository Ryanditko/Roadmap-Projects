# Replicação de Dados Multi-Região

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete e construa um sistema que replica dados entre regiões geográficas para que sobrevivam a uma queda de região inteira e sirvam leituras perto dos usuários — enfrentando o fato de que você não pode ter consistência forte, baixa latência e tolerância a partições ao mesmo tempo. É aqui que CAP e PACELC deixam de ser curiosidade e passam a ditar seu design. Você escolherá um modelo de consistência (replicação forte síncrona vs eventual assíncrona, ou algo no meio), definirá sua história de falha (RPO — quanto dado você pode perder; RTO — quão rápido você recupera), e tratará a realidade bagunçada de escritas concorrentes em duas regiões produzindo conflitos. Regras de residência de dados adicionam uma restrição que engenharia pura não descarta. A entrega é um design de replicação mais um protótipo funcional demonstrando failover e uma estratégia de resolução de conflitos documentada.

## Pré-requisitos

- Um datastore com recursos de replicação entre regiões (um DB distribuído, Kafka MirrorMaker ou replicação entre regiões de object store)
- Domínio sólido de modelos de consistência (forte, eventual, causal) e dos teoremas CAP/PACELC
- Entendimento dos conceitos de RPO/RTO e failover
- Familiaridade com resolução de conflitos (last-write-wins, vector clocks, CRDTs)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher um modelo de consistência e justificá-lo contra necessidades de latência e disponibilidade
- Definir e medir RPO e RTO para um cenário de perda de região
- Projetar e executar um failover (e failback) sem perda de dados além do seu RPO
- Resolver conflitos de escrita concorrente entre regiões com uma estratégia documentada
- Considerar restrições de residência de dados na topologia de replicação

## Requisitos Funcionais

1. Dados escritos em uma região devem replicar para ao menos outra região dentro de um lag limitado.
2. O sistema deve definir um modelo de consistência explícito e impô-lo consistentemente nas leituras.
3. Uma queda de região simulada deve disparar failover para outra região com perda de dados dentro do RPO declarado.
4. Escritas concorrentes na mesma chave em duas regiões devem ser reconciliadas por uma regra de resolução de conflitos documentada.
5. O lag de replicação e a saúde por região devem ser observáveis como métricas.
6. A topologia deve respeitar uma regra de residência de dados (ex.: dados de origem UE não saem de regiões da UE).

## Marcos Sugeridos

1. **Marco 1 — Replicar:** Suba duas regiões e replicação assíncrona; meça o lag de replicação base.
2. **Marco 2 — Failover e RPO/RTO:** Simule uma queda de região, faça failover, meça o RPO/RTO real, e então faça failback.
3. **Marco 3 — Conflitos e residência:** Force escritas conflitantes concorrentes, aplique uma estratégia de resolução e imponha uma restrição de residência.

## Esboço de Dados e Interface

```text
        região A (primária)               região B (réplica/ativa)
        [escrita] ──replica async──▶ lag de replicação = L
           │                                │
        leituras (fortes aqui)          leituras (eventuais aqui, salvo se promovida)

escolha de consistência (PACELC):
  se Partição -> escolha A (disponibilidade) ou C (consistência)
  Senão (operação normal) -> escolha L (latência) ou C (consistência)

failover: A caiu -> promover B ; RPO = dados escritos em A ainda não replicados
                                 RTO = tempo até B servir escritas
conflito (mesma chave, ambas regiões escrevem):
  estratégia A: last-write-wins por timestamp  (simples, pode perder escrita)
  estratégia B: version vectors / CRDT          (mescla, mais complexa)

residência: marque data{região_de_origem}; replique origem-UE só para regiões UE.
métricas: replication_lag_ms, saúde_da_região, contagem_de_conflitos.
```

## Desafios Extras

- Implemente escritas ativo-ativo em ambas as regiões com mescla baseada em CRDT e prove a convergência.
- Adicione failover automático com promoção guiada por health-check em vez de intervenção manual.
- Modele replicação baseada em quorum (escrever em W de N regiões) e analise seu perfil de RPO/latência.

## Definição de Pronto

- [ ] Escritas replicam entre regiões dentro de um lag medido e limitado.
- [ ] Uma simulação de queda de região faz failover com perda de dados dentro do RPO declarado e recuperação dentro do RTO.
- [ ] Escritas conflitantes concorrentes são reconciliadas por uma regra documentada e testada.
- [ ] O lag de replicação e a saúde da região são exportados como métricas.
- [ ] Uma restrição de residência é imposta e demonstrada (dados restritos nunca deixam suas regiões permitidas).

## Armadilhas Comuns

- Alegar "fortemente consistente e altamente disponível entre regiões" — o teorema CAP diz para escolher dois sob partição.
- Nunca testar failover de verdade, então o RTO é um chute e o runbook é ficção.
- Last-write-wins com relógios não sincronizados, descartando silenciosamente a escrita "perdedora".
- Ignorar residência até que um auditor encontre dados da UE replicados para us-east-1.

## Recursos

- [Modelos de consistência e CAP (Jepsen)](https://jepsen.io/consistency) — um mapa preciso das garantias de consistência.
- [Teorema PACELC](https://en.wikipedia.org/wiki/PACELC_theorem) — o tradeoff latência-vs-consistência mesmo sem partições.
- [Kafka: Geo-replicação (MirrorMaker 2)](https://kafka.apache.org/documentation/#georeplication) — replicação entre clusters/regiões na prática.
- [Amazon: Dynamo (artigo)](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — consistência eventual e resolução de conflitos em escala.
