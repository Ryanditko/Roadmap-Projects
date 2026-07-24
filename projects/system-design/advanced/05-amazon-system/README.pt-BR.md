# Projete uma Plataforma de E-Commerce como a Amazon

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete uma plataforma de e-commerce de larga escala no estilo da Amazon: um catálogo de centenas de milhões de produtos, busca full-text com ranqueamento por relevância, um carrinho de compras que sobrevive a sessões, um sistema de estoque que nunca vende além do disponível e um pipeline de pedidos que permanece correto sob carga de Black Friday. A tensão de design está entre uma experiência de navegação otimizada para leitura e um checkout fortemente consistente, onde dinheiro e contagens de estoque precisam ser exatos. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Entendimento sólido de modelos de dados relacionais vs NoSQL e quando cada um se encaixa
- Familiaridade com motores de busca (índice invertido, ranqueamento por relevância)
- Entendimento de transações, níveis de isolamento e idempotência
- Exposição a padrões orientados a eventos / saga para fluxos multi-etapa

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Particionar um catálogo e índice de busca entre shards mantendo as consultas rápidas
- Projetar reserva de estoque que previne venda em excesso sob concorrência
- Modelar um pedido como uma saga entre carrinho, pagamento, estoque e fulfillment
- Escolher consistência por domínio: eventual para navegação, forte para checkout/estoque
- Planejar tráfego de evento de pico (Black Friday) de 10–50× a linha de base

## Requisitos e Restrições

1. Servir páginas de produto e busca com latência p99 abaixo de ~300 ms em regime estável.
2. Nunca vender em excesso: uma unidade reservada para um pedido não pode ser vendida a outro.
3. Processar pedidos exatamente uma vez, mesmo quando clientes e serviços retentam.
4. Suportar ~300M de produtos e tráfego de pico de dezenas de milhões de requisições/minuto.
5. Manter o conteúdo do carrinho durável entre dispositivos e sessões.
6. Isolar a consistência do checkout da disponibilidade da navegação (navegação degrada, checkout permanece correto).
7. Suportar vendedores terceiros com estoque e preços independentes.

## Abordagem Sugerida

Separe fortemente os caminhos de leitura e escrita. O plano de navegação/busca é otimizado para leitura: documentos de produto desnormalizados em um índice de busca mais um cache, tolerante à consistência eventual. O plano de checkout é transacional: reserve estoque com uma escrita condicional (concorrência otimista ou um registro de reserva com TTL), depois rode o pedido como uma saga com ações compensatórias. Estime a vazão de pico e projete o store de reservas como o gargalo de escala que você protege. Use chaves de idempotência na submissão do pedido. Para a Black Friday, pré-compute páginas de produtos quentes, descarte carga não crítica e enfileire escritas para que o store de estoque degrade graciosamente em vez de vender em excesso.

## Esboço de Arquitetura

```text
Navegação: cliente ──> API ──> Svc Busca (shards de índice) + Cache de produto ──> BD Catálogo (fonte da verdade)
Carrinho:  cliente ──> Svc Carrinho ──> store de carrinho durável (por usuário)

Checkout (saga):
  submit(order, idempotencyKey)
    -> Svc Estoque: reservar (condicional/TTL)  --falha--> rejeita
    -> Svc Pagamento: autorizar (idempotente)   --falha--> libera reserva
    -> Svc Pedido: criar ORDER (PLACED)
    -> Svc Fulfillment: pick/pack/ship async
  compensação em qualquer falha reverte etapas anteriores

APIs principais:
GET  /search?q=...&filters=...     -> { results[], facets }
POST /cart/items                   -> { cart }
POST /orders (Idempotency-Key)     -> { orderId, status: PLACED }

Modelo de dados (esboço):
Product{ id, sellerId, attrs, priceCents }            # eventual para navegação
Inventory{ sku, available, reserved }                 # forte, escritas condicionais
Order{ id, userId, items[], state, idempotencyKey }   # dirigido por saga
```

## Tópicos de Aprofundamento

- **Concorrência de estoque:** reservas, locking otimista vs pessimista, limpeza por TTL.
- **Saga de pedido:** transações compensatórias, exactly-once via chaves de idempotência.
- **Sharding e relevância de busca:** particionamento de índice, sinais de ranqueamento, facetamento.
- **Divisão de consistência:** cache de navegação eventual vs checkout fortemente consistente.
- **Escala de evento de pico:** descarte de carga, enfileiramento de escritas, pré-computação de páginas quentes.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com os planos de navegação e checkout separados, refinado.
- [ ] Estimativas de capacidade para tamanho de catálogo, QPS de busca e vazão de checkout no pico.
- [ ] Os mecanismos de reserva de estoque e saga de pedido totalmente especificados.
- [ ] Uma análise de falha/DR: queda de pagamento no meio da saga, partição do store de estoque, estouro de cache.
- [ ] Um plano de Black Friday: o que degrada, o que permanece fortemente consistente e por quê.

## Armadilhas Comuns

- Decrementar o estoque só no momento do pagamento, permitindo que dois pedidos vendam a última unidade em excesso.
- Fazer o checkout inteiro uma transação distribuída gigante em vez de uma saga compensável.
- Servir o checkout do cache de navegação eventualmente consistente, mostrando preços/estoque desatualizados.
- Sem chave de idempotência na submissão do pedido, uma retentativa cria pedidos e cobranças duplicados.
- Tratar a Black Friday como "só mais tráfego" sem descarte de carga ou enfileiramento de escritas.

## Recursos

- [Dynamo: o Key-value Store Altamente Disponível da Amazon](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — o artigo canônico sobre disponibilidade e consistência eventual na Amazon.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de sharding, cache e consistência.
- [Padrão Saga (microservices.io)](https://microservices.io/patterns/data/saga.html) — transações compensatórias para fluxos multi-serviço.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — níveis de isolamento e transações distribuídas.
- [Amazon Builders' Library](https://aws.amazon.com/builders-library/) — práticas reais de confiabilidade e escala.
