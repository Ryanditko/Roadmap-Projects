# Deploy Multi-Região

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Atenda usuários a partir de mais de uma região geográfica para que uma queda regional não vire uma queda global, e para que um usuário em São Paulo não pague uma viagem transatlântica de ida e volta a cada requisição. Você vai implantar a mesma aplicação em pelo menos duas regiões, rotear usuários para a região saudável mais próxima e decidir — deliberadamente — como o estado é tratado entre elas. As decisões genuinamente difíceis vivem na camada de dados: você roda ativo-ativo com um store replicado globalmente aceitando consistência eventual, ou ativo-passivo com um failover claro e um ponto de recuperação aceito? Você também vai enfrentar residência de dados: alguns dados legalmente não podem deixar sua região. Este projeto é sobre tornar esses trade-offs explícitos e demonstráveis, não sobre copiar e colar uma stack em duas nuvens.

## Pré-requisitos

- Conforto para implantar uma aplicação em uma única região (contêineres ou VMs)
- Entendimento de DNS, TLS e como funciona o balanceamento de carga global
- Familiaridade com conceitos de replicação de banco de dados e modelos de consistência
- Consciência de latência, RPO e RTO como quantidades mensuráveis

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implantar uma pegada idêntica da aplicação em múltiplas regiões
- Rotear usuários para a região saudável mais próxima e fazer failover quando uma degradar
- Escolher e justificar uma estratégia de dados ativo-ativo vs. ativo-passivo
- Raciocinar sobre consistência, RPO e RTO para estado entre regiões
- Tratar restrições de residência de dados no roteamento e no armazenamento

## Requisitos Funcionais

1. A aplicação deve rodar em pelo menos duas regiões servindo a mesma funcionalidade.
2. Usuários devem ser roteados para a região saudável mais próxima (baseado em latência ou geo).
3. Se uma região ficar não saudável, o tráfego deve migrar para outra região automaticamente.
4. A estratégia de dados (replicação ou partição) deve ser explícita com um modelo de consistência declarado.
5. O sistema deve definir e medir RPO e RTO para uma falha regional.
6. Dados restritos por região nunca devem ser servidos de ou armazenados em uma região não permitida.
7. O failback para uma região recuperada deve ser controlado, não uma virada total instantânea.

## Marcos Sugeridos

1. **Marco 1 — Duas regiões no ar:** Implante stacks idênticas em duas regiões atrás de um ponto de entrada global.
2. **Marco 2 — Roteamento geo e saúde:** Roteie por proximidade e faça health-check de cada região.
3. **Marco 3 — Estratégia de dados:** Implemente replicação ou particionamento; documente consistência, RPO, RTO.
4. **Marco 4 — Simulação de failover e residência:** Simule uma queda regional, meça a recuperação e imponha regras de residência.

## Esboço de Dados e Interface

```text
                    ┌──────────────────────────┐
     usuários ─────▶│  DNS Global / LB Anycast  │  (roteamento latência/geo + saúde)
                    └───────┬───────────┬──────┘
                            ▼           ▼
                   ┌───────────┐   ┌───────────┐
                   │ Região A  │   │ Região B  │
                   │ (us-east) │   │ (sa-east) │
                   │  app+cache│   │  app+cache│
                   └─────┬─────┘   └─────┬─────┘
                         │ camada de dados│
              ativo-ativo │              │ ativo-passivo
              (replicado, ◀──────────────▶ (primário/réplica,
               eventual)     replicação    promove no failover)

Regra de residência (exemplo):
  dados marcados region=BR  -> armazenados/servidos só de sa-east

Metas não-funcionais:
  disponibilidade  >= 99,99% globalmente (sobrevive à perda de 1 região)
  RTO              < 5 min para mover tráfego + promover dados
  RPO              <= atraso de replicação (declare e defenda)
  orçamento de latência entre regiões declarado por caminho de requisição
```

## Desafios Extras

- Adicione uma terceira região e teste escritas por quórum ou padrões de leitura-local/escrita-global.
- Automatize o failback com uma reentrada canário para evitar que uma região fria pegue carga total.
- Adicione rastreamento de custo por região e roteie ciente de custo quando os SLOs permitirem.
- Rode um game day agendado de failover regional e publique o RTO/RPO medidos.

## Definição de Pronto

- [ ] O app atende tráfego de duas ou mais regiões com roteamento por proximidade.
- [ ] Uma queda regional simulada faz failover dentro do RTO declarado.
- [ ] A estratégia de dados está documentada com modelo de consistência, RPO e RTO explícitos.
- [ ] Dados restritos por residência comprovadamente nunca são servidos de uma região não permitida.
- [ ] O failback é controlado e observável.

## Armadilhas Comuns

- Implantar em duas regiões mas apontar ambas para o banco de uma só região — sem isolamento real.
- Ignorar o atraso de replicação e então perder dados no failover porque o RPO nunca foi medido.
- Assumir que o failover de DNS é instantâneo; caches e TTLs fazem dele qualquer coisa menos isso.
- Tratar residência de dados como um pensamento tardio e descobrir uma violação de conformidade em prod.
- Nunca testar o failover de verdade, então o sistema "multi-região" falha quando uma região falha.

## Recursos

- [AWS Well-Architected: Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html) — padrões de confiabilidade multi-região.
- [Google SRE Book: Managing Critical State](https://sre.google/sre-book/managing-critical-state/) — trade-offs de consistência e replicação.
- [Jepsen: modelos de consistência](https://jepsen.io/consistency) — um mapa preciso das garantias de consistência.
- [Cloudflare: o que é Anycast?](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/) — como o roteamento global direciona usuários.
