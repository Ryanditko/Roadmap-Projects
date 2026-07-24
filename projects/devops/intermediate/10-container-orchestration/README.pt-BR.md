# Orquestração de Contêineres

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Rode uma carga de trabalho real através de um pequeno cluster de máquinas e deixe um orquestrador — Kubernetes ou Docker Swarm — decidir onde os contêineres vão, manter o número desejado rodando e reagendá-los quando um nó morre. Este projeto te leva de "docker run em uma máquina" para um deploy declarativo e auto-recuperável. Você vai declarar um estado desejado, observar o orquestrador reconciliar a realidade em direção a ele, expor serviços através de rede e descoberta de cluster, anexar armazenamento persistente a cargas com estado e implantar uma nova versão sem downtime. A lição é a mentalidade do laço de reconciliação: você descreve o que quer, o control plane fecha continuamente a lacuna e seu trabalho passa de rodar comandos para escrever um estado desejado correto.

## Pré-requisitos

- Conforto para construir e rodar imagens de contêiner
- Dois ou mais nós (VMs, instâncias na nuvem ou um cluster multi-nó local como kind/k3d)
- Entender o básico de rede de contêineres (portas, DNS, overlays)
- Familiaridade com health checks e a diferença entre liveness e readiness
- Um trampolim: [Sistema de Balanceamento de Carga](../09-load-balancing/) explica o roteamento de serviços do qual isto depende

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Expressar uma carga de trabalho como estado desejado declarativo e deixar o orquestrador reconciliá-la
- Usar descoberta de serviços e rede de cluster para conectar componentes sem IPs fixos
- Anexar volumes persistentes para que contêineres com estado sobrevivam ao reagendamento
- Definir requests/limits de recursos e entender como o escalonador posiciona as cargas
- Executar uma atualização gradual (rolling update) e um rollback sem downtime

## Requisitos Funcionais

1. Um cluster multi-nó deve rodar uma carga com ao menos dois serviços que se comunicam.
2. A carga deve ser declarada como estado desejado (contagem de réplicas, imagem, recursos), não comandos imperativos.
3. Encerrar um contêiner ou drenar um nó deve disparar reagendamento automático para restaurar o estado desejado.
4. Os serviços devem se alcançar via descoberta de serviços, não endereços fixos.
5. Ao menos um serviço deve usar um volume persistente que sobreviva a um reagendamento.
6. Uma atualização gradual deve implantar uma nova versão sem perder tráfego, e o rollback deve ser possível.
7. Requests/limits de recursos devem ser definidos para que o escalonador posicione e proteja as cargas.

## Marcos Sugeridos

1. **Marco 1 — Cluster e deploy:** Suba o cluster e implante um serviço replicado a partir do estado desejado.
2. **Marco 2 — Rede e armazenamento:** Conecte a descoberta de serviços entre componentes e anexe um volume persistente.
3. **Marco 3 — Resiliência e rollout:** Comprove a auto-recuperação na perda de nó e faça um rolling update com rollback.

## Esboço de Dados e Interface

```text
cluster
  control-plane        reconcilia desejado vs real
  nó A, nó B ...        rodam contêineres agendados

estado desejado (por serviço)
  name           string
  image          repo:tag
  replicas       int
  resources      { requests, limits }
  probes         { liveness, readiness }
  network        nome do serviço -> endpoint virtual estável
  storage        reivindicação de volume (para stateful)

laço de reconciliação:
  observar real -> diff contra desejado -> agir (criar/matar/mover) -> repetir
  nó fora -> seus contêineres reagendados em nós saudáveis

rolling update:
  subir tag da imagem -> novas réplicas up + ready -> antigas drenadas -> repetir
  falha -> rollback para o estado desejado anterior
```

## Desafios Extras

- Adicione um ingress/gateway para que o tráfego externo alcance serviços por hostname ou caminho.
- Adicione autoescalonamento de réplicas com base em uma métrica (integra com o projeto de auto-escalonamento).
- Adicione gestão de config e secrets injetados nos contêineres em tempo de execução.
- Introduza regras de afinidade/anti-afinidade para que réplicas se espalhem por nós e zonas.

## Definição de Pronto

- [ ] O cluster roda a carga declarada na contagem de réplicas solicitada distribuída entre os nós.
- [ ] Encerrar um contêiner ou nó restaura o estado desejado automaticamente sem intervenção manual.
- [ ] Os serviços se comunicam via descoberta, sobrevivendo a reinícios e reagendamentos de contêineres.
- [ ] Um serviço com estado mantém seus dados através de um reagendamento via um volume persistente.
- [ ] Uma atualização gradual entrega uma nova versão sem requisições perdidas, e o rollback funciona.

## Armadilhas Comuns

- Tratar o orquestrador de forma imperativa (`run`/`kill` manual) e lutar contra o laço de reconciliação.
- Faltar readiness probes, fazendo o rolling update enviar tráfego a contêineres que ainda não estão prontos.
- Assumir que o disco local persiste — sem um volume real, os dados somem no reagendamento.
- Sem requests de recursos, deixando uma carga sufocar outras e causando despejos de vizinho barulhento.
- Fazer rollout com uma única réplica, de modo que a própria atualização é a queda.

## Recursos

- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/) — estado desejado, controladores e o modelo de reconciliação.
- [Kubernetes: Rolling Update Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment) — rollouts sem downtime e rollback.
- [Visão geral do Docker Swarm mode](https://docs.docker.com/engine/swarm/) — um orquestrador mais leve com as mesmas ideias centrais.
- [The Twelve-Factor App](https://12factor.net/) — config, ausência de estado e descartabilidade que a orquestração pressupõe.
