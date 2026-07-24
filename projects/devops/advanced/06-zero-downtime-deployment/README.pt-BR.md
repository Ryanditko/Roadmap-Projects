# Sistema de Deploy Sem Downtime

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Publique novas versões de um serviço enquanto ele atende tráfego ao vivo, sem derrubar uma única requisição. Você vai construir um sistema de deploy que move o tráfego gradualmente para a nova versão, observa sinais reais enquanto faz isso e reverte automaticamente no instante em que esses sinais pioram. A ideia central é transformar um release em um experimento controlado em vez de um salto de fé: um canário pega uma fatia do tráfego, métricas de saúde e SLO atuam como portões de promoção, e só um canário saudável ganha mais tráfego. Você também vai enfrentar as partes que silenciosamente fazem deploys "sem downtime" derrubarem requisições mesmo assim — drenagem de conexões, portões de readiness e mudanças de schema retrocompatíveis. A entrega é um pipeline repetível em que um deploy ruim é um não-evento.

## Pré-requisitos

- Um serviço containerizado rodando atrás de um load balancer ou ingress
- Observabilidade que expõe latência e taxa de erro como SLIs consultáveis
- Familiaridade com a mecânica de rollout do Kubernetes ou o equivalente da sua plataforma
- Entendimento de probes de readiness/liveness e do ciclo de vida de conexões

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar deploys canário e/ou blue-green com deslocamento gradual de tráfego
- Condicionar a promoção a métricas reais de SLI, não apenas "os pods estão de pé"
- Reverter automaticamente quando uma métrica de guarda de saúde ou SLO for violada
- Drenar conexões e usar portões de readiness para não derrubar requisições em andamento
- Tornar mudanças de schema e API retrocompatíveis entre versões

## Requisitos Funcionais

1. Uma nova versão deve receber tráfego gradualmente, começando por uma pequena porcentagem.
2. A promoção para mais tráfego deve ser condicionada a SLIs medidos (taxa de erro, latência).
3. Se uma métrica de guarda for violada, o sistema deve reverter automaticamente.
4. Nenhuma requisição em andamento pode ser derrubada durante um deslocamento ou rollback (drenagem de conexões).
5. A readiness deve condicionar o tráfego para que um pod receba requisições só quando realmente pronto.
6. Um rollback deve retornar à última versão boa conhecida sem cirurgia manual de manifest.
7. Todo deploy deve ser observável: split atual, saúde do canário e decisão.

## Marcos Sugeridos

1. **Marco 1 — Deslocamento gradual:** Implante uma nova versão em uma pequena fatia de tráfego atrás do LB.
2. **Marco 2 — Portões de promoção:** Consulte SLIs e promova só quando o canário estiver saudável.
3. **Marco 3 — Rollback automático:** Viole uma guarda de propósito e confirme o rollback automático.
4. **Marco 4 — Detalhes de segurança:** Adicione drenagem de conexões, portões de readiness e um passo a passo de mudança de schema retrocompatível.

## Esboço de Dados e Interface

```text
   deploy v2
      │
      ▼
   ┌──────────────┐   passo 1: 5%  ┌───────────────┐
   │ Controlador   │──────────────▶│ Split de tráf.│
   │ de rollout    │   passo 2: 25%│  v1 ▓▓▓░  v2 ░ │
   │ (Argo Rollouts│   passo 3: 50%└──────┬────────┘
   │  / Flagger)   │   passo 4:100%       │
   └──────┬────────┘                      ▼
          │  consulta SLIs        ┌───────────────┐
          └──────────────────────▶│ Métricas (SLI)│
          portão: promove se sã   │ erro%, latência│
          reverte se violado      └───────────────┘

Decisão de release por passo:
  se taxa_erro < 1% E p99 < 300ms por T minutos -> promove
  senão -> reverte para v1, drena conexões da v2

Metas não-funcionais:
  perda de requisição  0 requisições derrubadas durante deslocamento/rollback
  MTTR de rollback     < 60s da violação à reversão total
  disponibilidade      mantida >= SLO durante todo o deploy
```

## Desafios Extras

- Adicione feature flags para que o código seja publicado "no escuro" e habilitado independente do deploy.
- Suporte blue-green além do canário e compare seus trade-offs na sua carga de trabalho.
- Adicione análise automatizada (múltiplas métricas, comparação estatística com baseline) para promoção.
- Integre o rollout ao GitOps para que a versão desejada viva no Git e o rollout seja declarativo.

## Definição de Pronto

- [ ] Uma nova versão é promovida gradualmente com passos condicionados por SLI.
- [ ] Uma versão deliberadamente ruim é revertida automaticamente dentro do MTTR declarado.
- [ ] Nenhuma requisição é derrubada durante deslocamento ou rollback (verificado sob carga).
- [ ] A readiness condiciona o tráfego; a drenagem de conexões está no lugar.
- [ ] Uma mudança de schema retrocompatível é demonstrada entre duas versões.

## Armadilhas Comuns

- Tratar "pods rodando" como "saudável" e promover um canário que retorna erros rapidamente.
- Pular a drenagem de conexões, de modo que o rollback derruba justamente as requisições que você protegia.
- Quebrar a compatibilidade de schema, de modo que v1 e v2 não conseguem coexistir durante o deslocamento.
- Nenhum rollback automático, deixando um humano notar a violação minutos dentro de um incidente.
- Canário por contagem de pods em vez de porcentagem de tráfego, então o split não é o que você pensa.

## Recursos

- [Documentação do Argo Rollouts](https://argo-rollouts.readthedocs.io/) — canário e blue-green para Kubernetes.
- [Documentação do Flagger](https://docs.flagger.app/) — entrega progressiva com análise automatizada.
- [Martin Fowler: BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) — a descrição canônica do padrão.
- [Google SRE Workbook: Canarying Releases](https://sre.google/workbook/canarying-releases/) — prática de rollout progressivo seguro.
