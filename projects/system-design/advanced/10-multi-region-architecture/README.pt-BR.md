# Projete uma Arquitetura Multi-Região

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete um sistema que roda em múltiplas regiões geográficas para que usuários em qualquer lugar tenham baixa latência, e a perda de uma região inteira não derrube o produto. A parte difícil não é subir servidores em dois continentes — é a camada de dados. No momento em que você replica estado entre regiões, herda o trade-off do CAP: a replicação leva dezenas a centenas de milissegundos, então você precisa decidir, por tipo de dado, se tolera leituras desatualizadas, resolve conflitos de escrita ou paga a latência da consistência síncrona. Este é um exercício de design — o entregável é um documento de design com diagramas, não infraestrutura implantada.

## Pré-requisitos

- Entendimento do teorema CAP e modelos de consistência (forte, eventual, causal)
- Familiaridade com replicação de banco de dados (líder-seguidor, multi-líder, quórum)
- Conforto com DNS/balanceamento de carga global e roteamento baseado em saúde
- Estimativa básica de capacidade e custo entre regiões
- Um projeto intermediário de sistemas distribuídos como passo anterior (ex.: [Projete uma CDN](../../intermediate/10-cdn/))

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Estimar tráfego entre regiões, banda de replicação e o piso de latência imposto pela geografia
- Escolher um modelo de consistência por tipo de dado e justificá-lo frente ao custo de latência
- Projetar roteamento global de requisições com failover regional e health checks
- Projetar um plano de recuperação de desastres com metas explícitas de RTO e RPO
- Raciocinar sobre restrições de residência de dados/conformidade que fixam certos dados a uma região

## Requisitos e Restrições

**Funcionais**

- Servir leituras e escritas da região mais próxima do usuário quando possível.
- Replicar estado durável entre regiões e resolver escritas concorrentes conflitantes.
- Rotear usuários para uma região saudável e fazer failover automático quando uma fica indisponível.
- Suportar dados fixados a uma região para residência/conformidade (ex.: dados de usuário da UE permanecem na UE).

**Não-funcionais**

- Meta de disponibilidade 99,99%; sobreviver à perda de uma região inteira sem perda de dados críticos.
- Definir RTO (tempo para recuperar) e RPO (perda de dados aceitável) por classe de dado.
- Latência de leitura p99 dentro do orçamento regional; declare explicitamente a penalidade de escrita entre regiões.
- Consciente de custo: banda de replicação entre regiões e capacidade duplicada são os principais custos.

## Abordagem Sugerida

1. **Estime primeiro.** Escolha um cenário (ex.: 3 regiões, 100k escritas/seg globalmente, registro médio de 1 KB). Derive banda de replicação, RPS entre regiões e o piso de latência geográfica (RTT da velocidade da luz).
2. **Classifique os dados por necessidade de consistência.** Divida em forte (ex.: saldos de conta), eventual (ex.: perfis de usuário, feeds) e fixado à região (conformidade). Essa classificação guia toda decisão posterior.
3. **Escolha a topologia de replicação por classe.** Líder global único vs. multi-líder vs. quórum; esboce a resolução de conflitos (last-writer-wins, CRDTs ou merge no nível da aplicação).
4. **Projete o roteamento global.** GeoDNS ou Anycast mais health checks; defina o gatilho de failover e como escritas em trânsito são tratadas.
5. **Projete a DR.** Defina RTO/RPO por classe de dado, o runbook de failover e como testá-lo (game days).

## Esboço de Arquitetura

```text
                    Roteamento global (GeoDNS / Anycast + health checks)
                       │                                    │
          ┌────────────▼───────────┐          ┌─────────────▼──────────┐
          │   Região A (us-east)    │          │   Região B (eu-west)   │
          │  Camada de app          │          │  Camada de app         │
          │  Cache regional         │          │  Cache regional        │
          │  Réplica DB ◄───────────┼──replicação assíncrona/síncrona──┤
          └─────────────────────────┘          └────────────────────────┘
                       │  (escritas fixadas à região ficam locais por conformidade)
                    Região C (ap-south) ... mesmo formato

Registro replicado
  key:        string
  value:      bytes
  version:    vector clock | ts de Lamport  (para detecção de conflito)
  region:     id da região de origem
  residency:  GLOBAL | EU_ONLY | ...         (tag de conformidade)
  updatedAt:  epoch ms

Decisões de controle
  route(user)          -> região saudável mais próxima
  write(key, val)      -> líder local; replica conforme a classe de consistência
  onRegionDown(region) -> reroteia tráfego; promove réplica se o líder cair
  conflict(a, b)       -> resolve via LWW | merge CRDT | regra de app
```

## Tópicos de Aprofundamento

- **Modelos de consistência:** Forte vs. eventual vs. causal; onde cada um se encaixa e a latência que custa.
- **Resolução de conflitos:** Last-writer-wins vs. CRDTs vs. merge de aplicação — correção vs. complexidade.
- **Roteamento global e failover:** Efeitos do TTL de DNS na velocidade de failover; draining de Anycast; evitar split-brain.
- **RTO/RPO e testes de DR:** Definir metas por classe de dado e validá-las com game days.
- **Residência de dados:** Fixar dados a uma região e as implicações de roteamento para um usuário em viagem.

## Entregáveis

- Um documento de design (~4–6 páginas) cobrindo roteamento, replicação, consistência e DR.
- Estimativa de capacidade: banda de replicação entre regiões, RPS e o piso de latência geográfica, com premissas.
- Uma tabela de classificação de dados mapeando cada tipo de dado ao seu modelo de consistência e regra de residência.
- O diagrama de arquitetura, o modelo de dados replicado e o contrato de decisão do plano de controle.
- Uma seção de DR com metas explícitas de RTO/RPO por classe de dado e um esboço do runbook de failover.

## Armadilhas Comuns

- Aplicar um único modelo de consistência a todos os dados — pagando latência entre regiões em tudo ou corrompendo estado crítico.
- Ignorar o piso de latência da velocidade da luz; nenhuma engenharia torna escritas síncronas intercontinentais rápidas.
- Replicação multi-líder sem estratégia de resolução de conflitos, produzindo divergência silenciosa de dados.
- Definir RTO/RPO no papel mas nunca executar um drill de failover, deixando o runbook errado na hora que importa.
- Esquecer a lei de residência de dados; replicar dados de usuário da UE para outra região pode ser uma violação de conformidade.

## Recursos

- [System Design Primer: Teorema CAP](https://github.com/donnemartin/system-design-primer#cap-theorem) — o trade-off central consistência/disponibilidade.
- [Jepsen: Modelos de consistência](https://jepsen.io/consistency) — um mapa preciso de consistência forte a eventual.
- [AWS: Estratégias de recuperação de desastres](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html) — padrões de RTO/RPO e DR.
- [Amazon Aurora Global Database](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html) — referência concreta de replicação entre regiões.
- [CRDTs (tipos de dados replicados sem conflito)](https://crdt.tech/) — resolução de conflitos para escritas multi-líder.
