# Arquitetura Kubernetes Multi-Cluster

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Execute uma única plataforma lógica sobre vários clusters Kubernetes, de modo que a perda de um cluster — ou de uma zona de disponibilidade inteira — não derrube suas cargas de trabalho. Você vai subir pelo menos dois clusters, dar a eles uma identidade e uma estratégia de rede compartilhadas, e colocar na frente um plano de controle que decide onde as cargas rodam e como o tráfego chega até elas. Os problemas interessantes não são "instalar o Kubernetes duas vezes"; são descoberta de clusters, resolução de serviços entre clusters, propagação de configuração sem drift e um failover rápido o bastante para importar, mas conservador o bastante para não oscilar. Encare isto como um exercício de raciocínio sobre raio de impacto: o que quebra quando um cluster morre, e como você prova a recuperação de antemão?

## Pré-requisitos

- Experiência sólida com Kubernetes de cluster único — Deployments, Services, Ingress, RBAC ([trabalho com Kubernetes Deployment](../../intermediate/) é um bom degrau, se precisar)
- Conforto com um provedor de cluster gerenciado ou próprio (EKS, GKE, AKS ou kubeadm)
- Entendimento de DNS, balanceamento de carga e a distinção L4/L7
- Familiaridade com configuração declarativa e uma ferramenta como Helm ou Kustomize

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Desenhar uma topologia multi-cluster e justificar federação vs. clusters independentes
- Registrar clusters em um plano de controle central e propagar configuração de forma consistente
- Rotear tráfego entre clusters com failover ciente de saúde e preferência por localidade
- Replicar ou particionar estado deliberadamente, raciocinando sobre trade-offs de consistência
- Definir metas de disponibilidade e medir o tempo de recuperação em relação a elas

## Requisitos Funcionais

1. A plataforma deve rodar em pelo menos dois clusters com um único ponto de posicionamento de cargas.
2. Um serviço em um cluster deve ser resolvível e alcançável a partir de outro cluster.
3. Mudanças de configuração devem propagar para todos os clusters-alvo sem edição manual por cluster.
4. Quando um cluster ficar não saudável, o tráfego deve migrar para um cluster sobrevivente automaticamente.
5. O sistema deve expor saúde por cluster e agregada para que operadores vejam o raio de impacto.
6. Failover e failback devem ser reversíveis e não podem perder requisições em andamento silenciosamente.
7. Controle de acesso e políticas de segurança devem se aplicar de forma uniforme em todos os clusters.

## Marcos Sugeridos

1. **Marco 1 — Dois clusters, identidade compartilhada:** Provisione os clusters, estabeleça rede e uma raiz de confiança comum, verifique a alcançabilidade pod-a-pod entre clusters.
2. **Marco 2 — Posicionamento e propagação:** Introduza um plano de controle (Karmada, Cluster API ou ferramenta de fleet) que agenda cargas e sincroniza config nos dois clusters.
3. **Marco 3 — Tráfego entre clusters e failover:** Adicione resolução global de serviços e um roteador ciente de saúde, então mate um cluster e meça a recuperação.
4. **Marco 4 — Salvaguardas:** RBAC/network policy uniformes, observabilidade agregada e um runbook documentado de recuperação de desastres.

## Esboço de Dados e Interface

```text
                       ┌────────────────────────┐
  operadores / GitOps ─▶│  Plano de Controle /   │  (posicionamento, sync config)
                       │        Fleet            │
                       └───────────┬────────────┘
                        ┌──────────┴──────────┐
                        ▼                     ▼
                 ┌────────────┐        ┌────────────┐
   LB Global ───▶│ Cluster A  │        │ Cluster B  │◀─── LB Global
   (GeoDNS /     │  us-east   │◀──mTLS─▶│  eu-west   │
    Anycast)     └────────────┘        └────────────┘
                   svc: api    descoberta de serviço   svc: api
                                 entre clusters

Metas não-funcionais a declarar de antemão:
  disponibilidade  >= 99,95% na plataforma (sobrevive à perda de 1 cluster)
  RTO de failover  < 60s para tirar tráfego de um cluster morto
  drift de config  0 divergências não detectadas entre clusters
```

## Desafios Extras

- Adicione um terceiro cluster e teste políticas de posicionamento que respeitem localidade de região/zona.
- Introduza replicação de carga com estado (ex.: um datastore replicado) e raciocine sobre seu RPO.
- Automatize o failback com uma reentrada canário para que clusters recuperados não peguem carga total de imediato.
- Adicione consciência de custo para que o escalonador prefira clusters mais baratos quando os SLOs permitirem.

## Definição de Pronto

- [ ] Dois ou mais clusters rodam sob um único plano de controle com config sincronizada automaticamente.
- [ ] Um serviço é alcançável entre clusters e sobrevive à exclusão de um cluster.
- [ ] O failover ocorre dentro do seu RTO declarado e é observável em um dashboard.
- [ ] Não há drift de config não detectado — uma verificação sinaliza divergência.
- [ ] Existe um runbook de recuperação de desastres e ele foi ensaiado ao menos uma vez.

## Armadilhas Comuns

- Tratar multi-cluster como "prod + DR" e nunca exercitar o failover de verdade, então ele falha na hora que precisa.
- Sobrepor CIDRs de pod/service entre clusters, tornando o roteamento entre clusters impossível sem NAT.
- Deixar cada cluster derivar porque a config é aplicada manualmente em vez de reconciliada de uma fonte única.
- Replicar estado de forma ingênua e descobrir split-brain do jeito difícil durante uma partição.
- Ignorar o custo e o imposto operacional do cluster extra até a fatura ou o pager chegarem.

## Recursos

- [Kubernetes: conceitos multi-cluster](https://kubernetes.io/docs/concepts/cluster-administration/) — fundamentos de administração de cluster.
- [Documentação do Karmada](https://karmada.io/docs/) — um plano de controle de orquestração multi-cluster da CNCF.
- [Cluster API](https://cluster-api.sigs.k8s.io/) — gerenciamento declarativo do ciclo de vida de clusters.
- [Google SRE Book: Managing Critical State](https://sre.google/sre-book/managing-critical-state/) — trade-offs de consistência e failover.
