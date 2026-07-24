# Design de API Multi-Região

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Uma API de região única está a um incêndio de datacenter de distância de uma indisponibilidade global, e todo usuário do outro lado do planeta paga um imposto de latência em cada requisição. Este projeto pede que você projete e construa uma API que roda em duas ou mais regiões geográficas ao mesmo tempo: usuários são roteados para a região saudável mais próxima, os dados são replicados entre regiões e a perda de uma região inteira dispara um failover automático que o cliente mal percebe. A parte genuinamente difícil são os dados. No momento em que o mesmo registro pode ser escrito em duas regiões, você herda defasagem de replicação, escritas conflitantes e o trade-off do CAP na sua forma mais crua. Você escolherá uma topologia de replicação, decidirá que consistência pode honestamente prometer e projetará a resolução de conflitos — e então provará matando uma região sob carga.

## Pré-requisitos

- Experiência sólida construindo e implantando serviços HTTP com estado
- Entendimento do teorema CAP e modelos de consistência (forte, eventual, causal)
- Familiaridade com DNS, balanceamento de carga e health checks
- Conforto com conceitos de replicação de banco (líder/seguidor, multi-líder)
- Capacidade de rodar serviços em pelo menos dois ambientes isolados (regiões, VMs ou containers)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher uma topologia de replicação (líder único, multi-líder, sem líder) e justificá-la
- Rotear tráfego para a região saudável mais próxima com GeoDNS ou anycast e health checks
- Raciocinar honestamente sobre defasagem de replicação e que consistência você pode prometer
- Projetar e implementar resolução de conflitos para escritas concorrentes entre regiões
- Executar um failover regional automático e medir a janela de recuperação (RTO/RPO)
- Considerar restrições de residência de dados que prendem certos dados a uma região

## Requisitos Funcionais

1. A API deve rodar em pelo menos duas regiões, cada uma capaz de servir leituras e (conforme sua topologia) escritas de forma independente.
2. Clientes devem ser roteados para a região saudável mais próxima; uma região não saudável deve ser removida da rotação automaticamente.
3. Dados escritos em uma região devem replicar para as outras, e a defasagem de replicação deve ser observável.
4. O sistema deve definir seu modelo de consistência explicitamente (ex.: forte para escritas na região líder, eventual para leituras entre regiões) e se comportar de acordo.
5. Escritas conflitantes concorrentes devem ser resolvidas por uma estratégia documentada (last-writer-wins com ressalva, vetores de versão ou CRDTs) — não perdidas silenciosamente.
6. Perder uma região inteira deve disparar failover; requisições em voo e subsequentes devem ter sucesso contra uma região sobrevivente.
7. **Disponibilidade:** o design deve mirar um objetivo documentado de disponibilidade multi-região e sobreviver à perda de uma única região sem uma indisponibilidade total.
8. **Consistência vs. latência:** o trade-off deve ser explícito por endpoint; documente quais leituras podem estar defasadas e por quanto.
9. **Residência de dados:** o design deve suportar prender dados restritos a uma região (ex.: usuários da UE) e nunca replicá-los para fora de sua região permitida.

## Marcos Sugeridos

1. **Marco 1 — Duas regiões, leitura local:** Implante a API em duas regiões com um store de leitura compartilhado ou replicado; roteie por geografia.
2. **Marco 2 — Replicação:** Replique escritas entre regiões e exponha a defasagem de replicação; escolha líder único ou multi-líder.
3. **Marco 3 — Resolução de conflitos:** Force escritas conflitantes concorrentes e resolva-as pela estratégia escolhida.
4. **Marco 4 — Failover:** Adicione health checks e failover automático; mate uma região sob carga e meça RTO/RPO.

## Esboço de Dados e Interface

```text
Topologia

   usuários (UE) ─┐                                ┌─ usuários (US)
                  ▼                                 ▼
          [ GeoDNS / anycast + health checks ]
                  │                                 │
        ┌─────────▼───┐  replicação assíncrona ┌────▼────────┐
        │  Região UE  │◀──────────────────────▶│  Região US  │
        │ API + store │  (defasagem observável)│ API + store │
        └─────────────┘                        └─────────────┘
           │  dados presos por residência ficam só na região  │

Registro (para tratamento de conflito)
  id, value, version/vectorClock, region, updatedAt

Política de consistência (por endpoint)
  escritas: forte na região líder  |  leituras: local eventual (limite de defasagem)
Failover: health da região FALHA -> tira do DNS -> tráfego desloca -> meta de RTO
```

## Desafios Extras

- Adicione consistência read-your-writes para um usuário prendendo sua sessão a uma região.
- Implemente estado baseado em CRDT para um tipo de dado e compare com last-writer-wins.
- Adicione uma terceira região e observe como o custo de quórum/replicação muda.
- Construa um teste de caos que particiona regiões aleatoriamente e afirma que os invariantes se mantêm.

## Definição de Pronto

- [ ] Requisições são servidas pela região saudável mais próxima e fazem failover automaticamente quando ela morre.
- [ ] Uma escrita em uma região se torna visível nas outras dentro de uma defasagem medida e documentada.
- [ ] Escritas conflitantes concorrentes resolvem deterministicamente, sem perda silenciosa de dados.
- [ ] Uma região morta não causa indisponibilidade total; RTO e RPO são medidos e reportados.
- [ ] Dados presos por residência comprovadamente nunca deixam sua região permitida.
- [ ] Cada endpoint documenta sua garantia de consistência e a defasagem de pior caso.

## Armadilhas Comuns

- Assumir que replicação síncrona entre regiões é grátis — a velocidade da luz a torna uma assassina de latência; a maioria dos sistemas replica de forma assíncrona.
- "Last-writer-wins" com timestamps de relógio de parede entre regiões — a defasagem de relógio descarta silenciosamente a escrita errada.
- Fazer failover em um único health check falho, fazendo o tráfego oscilar entre regiões em oscilações transitórias.
- Replicar dados restritos por residência em todo lugar por conveniência, quebrando a conformidade.
- Prometer consistência forte em leituras entre regiões que são fisicamente eventuais, e depois depurar bugs "impossíveis".

## Recursos

- [Werner Vogels: Eventually Consistent](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html) — o ensaio fundacional sobre trade-offs de consistência em escala.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — replicação, conflitos multi-líder e modelos de consistência.
- [Jepsen: Consistency models](https://jepsen.io/consistency) — um mapa preciso do que cada nível de consistência de fato garante.
- [AWS: Multi-Region fundamentals](https://docs.aws.amazon.com/whitepapers/latest/aws-multi-region-fundamentals/aws-multi-region-fundamentals.html) — padrões para roteamento, replicação e failover.
- [CRDTs: Conflict-free Replicated Data Types (Shapiro et al.)](https://inria.hal.science/inria-00609399/document) — o artigo por trás dos tipos de dados que mesclam sem conflito.
