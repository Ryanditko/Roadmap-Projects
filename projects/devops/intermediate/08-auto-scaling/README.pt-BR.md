# Configuração de Auto-Escalonamento

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um auto-escalonador que adiciona e remove capacidade em resposta à carga, como faz um Auto Scaling Group na nuvem ou o Horizontal Pod Autoscaler do Kubernetes. Você vai coletar uma métrica (CPU, memória ou um sinal customizado como profundidade de fila ou requisições por segundo), compará-la com um alvo e decidir quantas instâncias o sistema deve rodar agora. As partes difíceis não são a aritmética, mas a teoria de controle ao redor: janelas de cooldown para não oscilar, limites mínimo e máximo para que uma métrica ruim não escale você a zero nem te leve à falência, e o tratamento honesto de cold starts, em que capacidade nova não é útil no instante em que aparece. Bem feito, o sistema acompanha a carga suavemente; feito de forma ingênua, ele oscila e te acorda às 3 da manhã.

## Pré-requisitos

- Conforto com uma fonte de métricas (Prometheus, CloudWatch ou um endpoint coletável)
- Uma carga de trabalho que você possa escalar — réplicas de um contêiner, VMs ou processos worker
- Entender o que "utilização" significa para a métrica escolhida
- Familiaridade com o padrão de laço de controle (observar → decidir → agir)
- Um trampolim: [Monitor de Reinício de Serviços](../../beginner/10-service-restart/) cobre o laço de health-check sobre o qual isto se apoia

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Selecionar e normalizar uma métrica de escalonamento e raciocinar por que ela reflete a carga real
- Implementar target-tracking: calcular réplicas desejadas a partir da utilização atual vs um alvo
- Aplicar comportamentos separados de scale-up e scale-down com cooldown para evitar flapping
- Impor limites min/max e proteções contra métricas ruins ou ausentes
- Considerar cold starts para que capacidade recém-adicionada não seja contada como produtiva cedo demais

## Requisitos Funcionais

1. O escalonador deve ler uma métrica em intervalo fixo de uma fonte real.
2. Deve calcular a contagem desejada de instâncias usando target-tracking (`desejado = atual * métrica / alvo`).
3. Scale-up e scale-down devem respeitar períodos de cooldown independentes.
4. A contagem de instâncias nunca deve sair do intervalo configurado `[min, max]`.
5. Métricas ausentes, obsoletas ou claramente inválidas não devem disparar escalonamento; o último estado bom se mantém.
6. Novas instâncias devem passar por uma checagem de readiness antes de contarem como capacidade.
7. Toda decisão de escalonamento (métrica, desejado, real, motivo) deve ser registrada para inspeção posterior.

## Marcos Sugeridos

1. **Marco 1 — Laço de métrica:** Colete uma métrica em intervalo e registre a utilização atual.
2. **Marco 2 — Target tracking:** Calcule réplicas desejadas e atue com scale-up/down dentro dos limites.
3. **Marco 3 — Estabilidade:** Adicione cooldowns, gate de readiness e proteções contra métricas ruins.

## Esboço de Dados e Interface

```text
política
  metric        cpu | memory | custom(name)
  target        número        (ex.: 60 para 60% de CPU)
  min, max      int
  cooldown      { up_s, down_s }
  step_max      int           (mudança máx por decisão)

laço de controle (a cada intervalo)
  1. ler métrica M para N instâncias atuais
  2. desejado = ceil(N * (M / target))
  3. limitar desejado a [min, max] e a +/- step_max
  4. se subindo e último-up dentro de up_s      -> segurar
     se descendo e último-down dentro de down_s -> segurar
  5. atuar; novas instâncias excluídas até ready

entrada do log de decisão
  ts, valor_métrica, N, desejado, aplicado, motivo
```

## Desafios Extras

- Adicione escalonamento preditivo a partir de uma tendência móvel para que a capacidade anteceda a carga em vez de persegui-la.
- Suporte múltiplas métricas e escale pela mais exigente.
- Adicione escalonamento agendado para picos diários conhecidos junto ao escalonamento reativo.
- Emita uma estimativa de custo por decisão para que o scale-up fique visivelmente ligado ao gasto.

## Definição de Pronto

- [ ] Um aumento sustentado de carga escala o sistema para cima em até uma janela de cooldown e para no máximo.
- [ ] A carga caindo o escala de volta para baixo, nunca abaixo do mínimo.
- [ ] Oscilações rápidas da métrica não causam flapping — o cooldown suprime a oscilação de forma demonstrável.
- [ ] Uma leitura de métrica ausente ou absurda mantém o estado em vez de escalar descontroladamente.
- [ ] Novas instâncias só contam quando prontas, e toda decisão é registrada com seu motivo.

## Armadilhas Comuns

- Escalar por uma métrica atrasada (ex.: CPU média) que reage devagar demais, então você sempre escala tarde.
- Cooldowns simétricos: descer com a mesma pressa com que sobe causa thrash sob tráfego em rajadas.
- Ignorar cold starts, fazendo o escalonador adicionar mais capacidade enquanto o último lote ainda aquece.
- Sem limite máximo, deixando uma métrica descontrolada ou uma queda do sistema de métricas escalar você a uma conta enorme.
- Confiar em uma única coleta; uma amostra ruim dispara um evento de escalonamento que a próxima amostra reverte.

## Recursos

- [Kubernetes: Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) — o algoritmo de target-tracking em detalhe.
- [AWS: Target Tracking Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html) — o modelo de um auto-escalonador de produção.
- [Prometheus: Querying Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/) — como puxar uma métrica para escalar.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — por que limites e comportamento gracioso importam sob carga.
