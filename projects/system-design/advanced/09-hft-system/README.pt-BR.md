# Projete um Sistema de Trading de Alta Frequência

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete o núcleo de um sistema de trading de alta frequência (HFT): o caminho que ingere um feed de dados de mercado, executa uma estratégia, passa por verificações de risco pré-negociação e submete uma ordem a uma bolsa — tudo em microssegundos de um único dígito. Nesse extremo do espectro de latência, os instintos usuais de design de sistemas se invertem. Pausas de garbage collection, stacks de rede do kernel e até cache misses tornam-se preocupações de design de primeira ordem, e a correção sob um orçamento estrito de risco importa tanto quanto a velocidade. Este é um exercício de design — o entregável é um documento de design rigoroso, não um motor de trading em execução.

## Pré-requisitos

- Domínio sólido de concorrência, estruturas de dados lock-free e layout de memória
- Entendimento de internals de rede (kernel bypass, NIC, TCP vs. UDP multicast)
- Familiaridade com a mecânica do livro de ofertas e tipos de ordem de bolsa
- Conforto com raciocínio de latência de cauda (p99/p99,9) e estimativa de capacidade
- Base intermediária em sistemas distribuídos (um passo anterior: [Projete um Cache Distribuído](../../intermediate/) ou similar)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Realizar estimativa de capacidade rigorosa: taxa de mensagens de mercado, taxa de ordens e orçamento de latência por estágio
- Projetar um caminho quente wire-to-wire que minimiza latência (kernel bypass, busy-polling, layout amigável a cache)
- Raciocinar sobre o trade-off consistência vs. latência no risco pré-negociação e no rastreamento de posição
- Projetar tratamento de falhas e recuperação de desastres que nunca deixe ordens em estado desconhecido
- Justificar onde gastar microssegundos e onde aceitá-los

## Requisitos e Restrições

**Funcionais**

- Ingerir um feed normalizado de dados de mercado e manter um livro de ofertas em memória.
- Executar uma estratégia que emite intenções de ordem em atualizações do livro.
- Aplicar verificações de risco pré-negociação (limites de posição, tamanho máximo de ordem, price collars, kill switch).
- Submeter, alterar e cancelar ordens no gateway da bolsa; reconciliar execuções (fills).
- Persistir um log de auditoria imutável e ordenado de cada decisão para conformidade.

**Não-funcionais**

- Orçamento de latência wire-to-wire p99 de ~5–10 µs para o caminho de decisão; declare a divisão por estágio.
- Sustentar picos de dados de mercado (projete para um burst de milhões de mensagens/seg).
- Sem pausas de GC ilimitadas no caminho quente; latência determinística é o objetivo, não apenas a média baixa.
- Verificações de risco devem estar corretas mesmo sob recuperação — nunca exceder um limite de posição.
- Recuperar de falha de processo/host com um estado de ordem conhecido e reconciliado.

## Abordagem Sugerida

1. **Estime o orçamento.** Escolha números concretos (ex.: 2M msgs/seg de pico de dados de mercado, 50k ordens/seg, alvo wire-to-wire de 8 µs). Aloque o orçamento de microssegundos entre parse → atualização do livro → estratégia → risco → envio.
2. **Projete o caminho quente.** NIC com kernel bypass (DPDK/Solarflare), busy-poll em vez de interrupções, ring buffers pré-alocados, pipeline sequenciado de único escritor. Mantenha a thread de decisão fixada a um core.
3. **Modele o livro de ofertas.** Estrutura de níveis de preço amigável a cache com acesso O(1) ao topo do livro.
4. **Projete o risco como um estágio rápido in-line.** Estado de posição em uma estrutura lock-free; decida quão forte precisa ser sua consistência versus o custo de latência.
5. **Projete failover e DR.** Log de eventos sequenciado, standby quente e um protocolo de reconciliação com a bolsa no restart.

## Esboço de Arquitetura

```text
Multicast da bolsa ─UDP─► Feed handler ─► Livro de ofertas (mem, cache-aligned)
   (dados de mercado)      (kernel bypass)         │
                                                   ▼
                                           Motor de estratégia
                                                   │ intenção de ordem
                                                   ▼
                                           Risco pré-negociação ─rejeita─► descarta + log
                                           (posição, tamanho, collar,
                                            kill switch)
                                                   │ aprova
                                                   ▼
                                           Gateway de ordens ─TCP/binário─► Bolsa
                                                   │
   Sequenciador ──► log append-only ──► standby quente (replay no failover)

Ordem (evento)
  seq:       uint64        (monotônico, único escritor)
  clientId:  uint64
  symbol:    uint32        (internado)
  side:      BUY | SELL
  price:     int64         (ticks de ponto fixo)
  qty:       uint32
  tsNanos:   uint64
  state:     NEW | ACK | PARTIAL | FILLED | CANCELED | REJECTED

API interna (in-process, sem rede no caminho quente)
  onMarketData(msg)     -> book.apply(msg)
  strategy.evaluate()   -> Optional<OrderIntent>
  risk.check(intent)    -> APPROVE | REJECT(reason)
  gateway.submit(order) -> ACK assíncrono via sequenciador
```

## Tópicos de Aprofundamento

- **Orçamento de latência:** Medir e atribuir latência wire-to-wire; por que p99,9 e jitter importam mais que a média.
- **Kernel bypass e busy-polling:** Trade-off de queimar um core por determinismo vs. I/O orientado a interrupções.
- **Consistência vs. latência no risco:** Verificações de posição fortes in-line vs. otimista-com-reconciliação, e os modos de falha de cada uma.
- **Memória determinística:** Object pools, alocação por arena e evitar GC no caminho quente.
- **Falha e DR:** Replicação baseada em sequenciador, reconciliação com a bolsa e semântica do kill-switch no restart.

## Entregáveis

- Um documento de design (~4–6 páginas) cobrindo o caminho quente, o risco e a recuperação.
- Uma estimativa de capacidade rigorosa: taxas de mensagens, taxas de ordens e um orçamento de latência por estágio em microssegundos com premissas.
- O diagrama de arquitetura, o modelo de evento/dados e o contrato da API interna.
- Uma análise explícita de trade-off consistência vs. latência para o estágio de risco, com a opção rejeitada e o porquê.
- Uma seção de falha/DR: o que acontece na perda de host e como o estado da ordem é reconciliado na recuperação.

## Armadilhas Comuns

- Otimizar a latência média ignorando latência de cauda e jitter — HFT vive e morre no p99,9.
- Colocar verificações de risco fora do caminho quente "por velocidade", criando uma janela onde um limite de posição pode ser violado.
- Assumir que o GC de um runtime é negligenciável; uma pausa não planejada é um mercado perdido ou uma ordem descontrolada.
- Projetar recuperação que faz replay de ordens sem reconciliar contra a bolsa, causando ordens duplicadas ou perdidas.
- Negligenciar o kill switch — todo design de HFT precisa de uma forma rápida e testada de parar todo o trading.

## Recursos

- [Artigo técnico do LMAX Disruptor](https://lmax-exchange.github.io/disruptor/disruptor.html) — o design canônico de ring-buffer de único escritor e baixa latência.
- [DPDK: Guia do Programador](https://doc.dpdk.org/guides/prog_guide/) — fundamentos de processamento de pacotes com kernel bypass.
- ["Latency Numbers Every Programmer Should Know"](https://gist.github.com/jboner/2841832) — base para o orçamento de microssegundos.
- [Nasdaq: Como funcionam os motores de matching](https://www.nasdaq.com/articles/how-do-exchanges-match-orders) — tipos de ordem e mecânica de matching.
- [System Design Primer: Latência vs. throughput](https://github.com/donnemartin/system-design-primer#latency-vs-throughput) — enquadramento do trade-off.
