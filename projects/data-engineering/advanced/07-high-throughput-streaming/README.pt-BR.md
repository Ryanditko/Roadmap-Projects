# Sistema de Streaming de Alta Vazão

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa e ajuste um sistema de streaming para mover milhões de eventos por segundo, e então meça-o honestamente. Este é um projeto de engenharia de performance: o objetivo não é uma nova funcionalidade, mas um benchmark documentado e reproduzível, e um conjunto de decisões de tuning que comprovadamente movem os números. Você empurrará contra a tensão fundamental entre vazão e latência — lotes maiores e compressão elevam a vazão mas adicionam latência; lotes menores cortam latência mas desperdiçam overhead por mensagem — e descobrirá onde seu sistema cai. Pelo caminho você tratará backpressure (o que acontece quando os consumidores não acompanham), escolherá um formato de serialização, ajustará particionamento e paralelismo, e separará gargalos genuínos do ruído. A entrega é um sistema mais um relatório de benchmark mostrando vazão, latência p50/p99 e o efeito de cada botão de tuning.

## Pré-requisitos

- Experiência com uma plataforma de streaming (Kafka, Pulsar, Redpanda) e seus botões de tuning de produtor/consumidor
- Conforto para perfilar e ler percentis de latência, não só médias
- Entendimento de tradeoffs de batching, compressão e serialização
- Familiaridade com conceitos de backpressure e controle de fluxo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um benchmark de vazão/latência repetível com um gerador de carga controlado
- Ajustar batching, compressão e contagem de partições e quantificar cada efeito
- Tratar backpressure para que um consumidor sobrecarregado degrade graciosamente em vez de colapsar
- Escolher um formato de serialização (Avro/Protobuf vs JSON) com base em custo medido
- Identificar o gargalo real (CPU, rede, GC, disco) em vez de adivinhar

## Requisitos Funcionais

1. Um gerador de carga deve produzir uma taxa de eventos sustentada e controlável para o benchmark.
2. O sistema deve reportar vazão (eventos/s e bytes/s) e percentis de latência de ponta a ponta (p50/p99).
3. O backpressure deve ser tratado: quando os consumidores atrasam, o sistema deve throttle ou bufferizar dentro de limites, nunca perder dados silenciosamente nem dar OOM.
4. Ao menos três botões de tuning (ex.: tamanho de lote, compressão, contagem de partições) devem ser variados e seus efeitos medidos.
5. O benchmark deve ser reproduzível: mesma config, mesmo resultado dentro da tolerância.
6. O sistema deve sustentar uma taxa alvo por uma janela sustentada sem crescimento ilimitado de lag.

## Marcos Sugeridos

1. **Marco 1 — Base e harness:** Construa o gerador de carga e a coleta de métricas; registre uma base sem tuning (vazão, p99).
2. **Marco 2 — Ajustar:** Varra batching, compressão, serialização e contagem de partições; plote o efeito de vazão/latência de cada botão.
3. **Marco 3 — Backpressure e limites:** Adicione controle de fluxo, então empurre além da capacidade para achar e documentar o ponto de ruptura.

## Esboço de Dados e Interface

```text
[gerador de carga] --taxa R--> [produtor]
   botões: batch.size, linger.ms, compression{none|lz4|zstd}, serialization{json|proto}
              │
              ▼
        [log: P partições]   vazão escala ~ com P (até certo ponto)
              │
              ▼
        [consumidores, C instâncias]   lag do consumidor = offset mais recente - offset commitado
              │
              ▼
        [sink]

backpressure: se lag > limite -> throttle produtor / crescer buffer até limite B
              nunca: descartar silenciosamente, ou bufferizar ilimitadamente -> OOM

relatório (por config):
  vazao_eps | vazao_MBps | p50_ms | p99_ms | lag_max | cpu% | gc_ms
tradeoff observado:  lote maior => maior vazão, maior latência p99
```

## Desafios Extras

- Adicione auto-scaling de consumidores guiado por lag e mostre que ele segura a latência sob um pico.
- Compare zero-copy / sem-compressão vs zstd e quantifique o tradeoff de CPU/rede.
- Perfile e elimine um pico de p99 causado por pausa de GC (ajuste o heap ou mude para buffers off-heap).

## Definição de Pronto

- [ ] O benchmark é reproduzível e reporta vazão mais latência p50/p99.
- [ ] Ao menos três botões de tuning são varridos com efeitos plotados e explicados.
- [ ] O backpressure mantém o sistema limitado sob sobrecarga — sem perda silenciosa, sem OOM.
- [ ] O ponto de ruptura documentado identifica o gargalo real (CPU/rede/GC/disco).
- [ ] O tradeoff vazão/latência é demonstrado com dados, não afirmado.

## Armadilhas Comuns

- Reportar latência média, escondendo a cauda p99 onde mora a dor real.
- Fazer benchmark com um gerador de carga fraco demais para saturar o sistema — você mede o gerador, não o pipeline.
- Adicionar partições além do ponto de retornos decrescentes e pagar overhead de coordenação à toa.
- "Consertar" backpressure com uma fila em memória ilimitada que só adia o crash.

## Recursos

- [Kafka: Configs de produtor e consumidor](https://kafka.apache.org/documentation/#producerconfigs) — botões de batching, linger e compressão.
- [Confluent: Otimizando vazão vs latência no Kafka](https://docs.confluent.io/cloud/current/client-apps/optimizing/throughput.html) — o tradeoff central, com configurações concretas.
- [Flink: Back Pressure](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/monitoring/back_pressure/) — como um processador expõe e trata sobrecarga.
- [Brendan Gregg: Percentis de latência e método USE](https://www.brendangregg.com/usemethod.html) — uma abordagem rigorosa para achar gargalos.
