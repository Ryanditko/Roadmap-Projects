# Deploy Blue/Green

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Implemente um deploy blue/green: rode dois ambientes de produção idênticos — "blue" (ao vivo) e "green" (ocioso) — implante a nova versão no ocioso, valide-a isoladamente e então vire um roteador ou load balancer para enviar todo o tráfego a ele em uma única troca atômica. Se algo estiver errado, volte instantaneamente. O valor é releases com zero downtime e um rollback instantâneo que não exige rebuild. Você vai encarar as partes difíceis que todos pulam: como verificar a saúde do ambiente ocioso antes da troca e o que fazer com mudanças de esquema de banco que ambas as versões precisam tolerar.

## Pré-requisitos

- Uma app implantável e um roteador/load balancer que você possa reprogramar (Nginx, HAProxy, LB de nuvem ou Service do Kubernetes)
- Dois ambientes que você consiga rodar em paralelo (contêineres, VMs ou namespaces)
- Entendimento de health checks e de como o tráfego é roteado a um backend
- Consciência de que o banco de dados costuma ser compartilhado entre blue e green

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Rodar dois ambientes paralelos, implantáveis de forma independente, atrás de um único ponto de entrada
- Implantar e validar uma nova versão sem nenhum tráfego de usuário chegando a ela
- Trocar todo o tráfego atomicamente e reverter trocando de novo
- Verificar a saúde do ambiente ocioso como portão antes da troca
- Raciocinar sobre mudanças de esquema que devem ser retrocompatíveis durante a sobreposição

## Requisitos Funcionais

1. Dois ambientes (blue e green) devem poder rodar simultaneamente atrás de um único roteador.
2. Uma nova versão deve implantar no ambiente ocioso enquanto o ao vivo continua servindo.
3. O ambiente ocioso deve passar em um health/smoke check antes de poder receber tráfego.
4. A troca de tráfego deve ser atômica — nenhuma requisição deve atingir um estado meio trocado.
5. O rollback deve ser uma única troca de volta ao ambiente anterior, sem rebuild.
6. Os usuários devem observar zero downtime e zero requisições falhas durante a troca.
7. O ambiente ativo (blue vs green) deve ser observável a qualquer momento.

## Marcos Sugeridos

1. **Marco 1 — Dois ambientes:** Rode blue e green atrás de um roteador; aponte-o para blue.
2. **Marco 2 — Implantar e validar o ocioso:** Implante em green, verifique sua saúde enquanto blue serve.
3. **Marco 3 — Trocar e reverter:** Vire o tráfego para green, valide, e então pratique um rollback instantâneo.

## Esboço de Dados e Interface

```text
Topologia:
                    ┌───────────┐
   clientes ──────> │ roteador  │ ──(ativo)──> BLUE  (v1, ao vivo)
                    └───────────┘        └────> GREEN (v2, ocioso, em validação)

Troca = reapontar o upstream do roteador de BLUE para GREEN (atômico)
Rollback = reapontar de volta para BLUE

Portão antes da troca:
  implanta v2 -> GREEN
  roda health/smoke checks contra GREEN diretamente (não via roteador)
  tudo ok? -> troca ; senão -> aborta, GREEN permanece ocioso

Ressalva do DB compartilhado:
  o esquema deve satisfazer AMBOS v1 e v2 durante a sobreposição (expand/contract)
```

## Desafios Extras

- Automatize toda a virada a partir de um pipeline, com o portão de saúde como passo obrigatório.
- Adicione um passo canário: roteie uma pequena porcentagem para green antes da troca total.
- Modele uma migração expand/contract para que uma mudança de esquema seja segura na troca.
- Mantenha o ambiente antigo aquecido por uma janela definida antes de reaproveitá-lo.

## Definição de Pronto

- [ ] Blue e green rodam lado a lado com versões independentes.
- [ ] Uma nova versão é validada no ambiente ocioso antes de qualquer tráfego chegar a ela.
- [ ] A troca causa zero requisições falhas (verificado com um gerador de carga durante a virada).
- [ ] O rollback é uma única retroca sem rebuild e conclui em segundos.
- [ ] O ambiente atualmente ativo é sempre identificável.

## Armadilhas Comuns

- Trocar antes de o ambiente ocioso estar realmente pronto, fazendo o release de "zero downtime" servir erros.
- Uma troca não atômica (ex.: editar config em vários LBs um a um) deixando uma janela de split-brain.
- Esquecer do banco compartilhado: uma mudança de esquema só-v2 quebra v1 no instante em que você precisaria reverter.
- Reaproveitar o ambiente antigo imediatamente, destruindo sua rede de segurança de rollback instantâneo.
- Drenar conexões abruptamente, cortando requisições em andamento no momento da troca.

## Recursos

- [Martin Fowler: BlueGreenDeployment](https://martinfowler.com/bliki/BlueGreenDeployment.html) — a descrição canônica do padrão.
- [AWS: Whitepaper de deploys Blue/Green](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/welcome.html) — técnicas e trade-offs.
- [Expand/contract (parallel change)](https://martinfowler.com/bliki/ParallelChange.html) — mudanças de esquema seguras entre versões.
- [Deployments do Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) — uma forma de modelar blue/green com Services.
</content>
