# Deploy no Kubernetes

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Pegue uma aplicação web em contêiner e rode-a corretamente em um cluster Kubernetes — não apenas "kubectl run e torça", mas um deploy de verdade com manifestos declarativos, um endereço de serviço estável, ingress externo, configuração separada dos segredos, requisições de recursos, sondas de saúde e uma atualização gradual que entrega uma nova versão sem derrubar o tráfego. Você pode usar qualquer cluster: um local como kind, k3d ou minikube, ou um cluster gerenciado na nuvem. A lição é como o Kubernetes transforma um conjunto de arquivos YAML de estado desejado em um sistema em execução autocurável e atualizável.

## Pré-requisitos

- Uma imagem de contêiner de uma app, publicada em um registro que o cluster consiga puxar
- Um cluster funcional e o `kubectl` configurado contra ele
- Entendimento de contêineres, portas e variáveis de ambiente
- Familiaridade com YAML e o ciclo requisição/resposta de uma app web

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Expressar o estado desejado com manifestos de Deployment, Service e Ingress
- Separar configuração (ConfigMap) de segredos e injetar ambos nos pods
- Definir requisições/limites de recursos e raciocinar sobre agendamento e evicção
- Configurar sondas de liveness e readiness para que o tráfego só atinja pods saudáveis
- Realizar uma atualização gradual e revertê-la quando a nova versão se comportar mal

## Requisitos Funcionais

1. A app deve ser descrita por um Deployment rodando ao menos duas réplicas.
2. Um Service deve dar aos pods um endereço estável dentro do cluster, independente da rotatividade de pods.
3. O tráfego externo deve chegar à app por meio de um Ingress (ou LoadBalancer equivalente).
4. Configuração não sensível deve vir de um ConfigMap; valores sensíveis, de um Secret.
5. Todo pod deve declarar requisições e limites de recursos.
6. Sondas de readiness e liveness devem manter o tráfego longe de pods que estão iniciando ou não saudáveis.
7. Uma atualização gradual deve entregar uma nova imagem com zero requisições perdidas, e `kubectl rollout undo` deve restaurar a versão anterior.

## Marcos Sugeridos

1. **Marco 1 — Rodar:** Deployment + Service, app acessível dentro do cluster.
2. **Marco 2 — Expor e configurar:** Adicione Ingress, ConfigMap e Secret; conecte a config aos pods.
3. **Marco 3 — Resiliência e atualizações:** Adicione sondas e limites de recursos; faça uma atualização gradual e um rollback.

## Esboço de Dados e Interface

```text
Relações entre objetos:
  Ingress ──roteia──> Service ──seleciona(labels)──> Pods (do Deployment)
  ConfigMap ─┐
  Secret ────┴─montado/env─> Pods

Spec do Deployment (só estrutura):
  replicas: 2
  strategy: RollingUpdate (maxUnavailable, maxSurge)
  template:
    containers:
      - image: registry/app:<tag>
        resources: { requests: {cpu, mem}, limits: {cpu, mem} }
        readinessProbe: httpGet /healthz
        livenessProbe:  httpGet /healthz
        envFrom: [configMapRef, secretRef]

Atualização: kubectl set image ...  -> novo ReplicaSet sobe, antigo desce
Rollback:    kubectl rollout undo deployment/app
```

## Desafios Extras

- Adicione um HorizontalPodAutoscaler que escala réplicas por CPU.
- Adicione um PodDisruptionBudget para que evicções voluntárias nunca levem a app abaixo de um piso.
- Divida em um StatefulSet um componente que precise de identidade estável e armazenamento.
- Adicione NetworkPolicies restringindo quais pods podem conversar entre si.

## Definição de Pronto

- [ ] Excluir um pod resulta no Kubernetes recriando-o automaticamente.
- [ ] O endereço do Service permanece estável enquanto os pods são substituídos.
- [ ] Config e segredos são injetados de ConfigMap/Secret, não embutidos na imagem.
- [ ] Uma atualização gradual conclui sem requisições falhas contra a app.
- [ ] `kubectl rollout undo` restaura a versão anterior e ela serve tráfego.

## Armadilhas Comuns

- Omitir sondas de readiness, fazendo o tráfego atingir um pod antes de ele poder servir e os usuários verem erros.
- Não definir requisições de recursos, deixando um pod barulhento sufocar seus vizinhos.
- Embutir segredos na imagem ou em um ConfigMap em vez de um Secret.
- Seletores de labels incompatíveis entre Deployment e Service, deixando o Service com zero endpoints.
- Supor que uma atualização gradual é segura sem uma sonda — o Kubernetes vai alegremente rotear para um pod novo quebrado.

## Recursos

- [Conceitos do Kubernetes](https://kubernetes.io/docs/concepts/) — o modelo de objetos, explicado pelo projeto.
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) — atualizações graduais e rollbacks.
- [Configurar sondas liveness, readiness e startup](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — semântica das sondas.
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) — roteando tráfego externo para dentro do cluster.
</content>
