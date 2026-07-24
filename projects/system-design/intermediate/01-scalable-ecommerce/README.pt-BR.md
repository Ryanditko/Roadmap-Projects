# Projete um Sistema de E-commerce Escalável

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de uma loja online como a Amazon ou a Shopify: navegar por um catálogo, adicionar itens ao carrinho, finalizar a compra e acompanhar pedidos. A parte difícil não é o CRUD — é manter o estoque consistente quando milhares de compradores disputam a última unidade de um item popular, mantendo o catálogo rápido de navegar. O tráfego de navegação quer velocidade e tolera dados desatualizados; o checkout quer correção e não pode vender mais do que existe. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas, não código funcional.

## Pré-requisitos

- Familiaridade com APIs HTTP e modelos de dados relacionais vs. NoSQL
- Entendimento básico de transações de banco de dados e ACID
- Noção conceitual de cache e filas de mensagens
- Conforto para raciocinar sobre trade-offs de consistência vs. disponibilidade

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Decompor um domínio em serviços (catálogo, carrinho, estoque, pedido, pagamento)
- Estimar QPS de leitura/escrita e armazenamento para um catálogo e fluxo de pedidos
- Escolher modelos de consistência por subsistema — eventual para navegação, forte para checkout
- Projetar um esquema de reserva de estoque que sobreviva a checkouts concorrentes
- Justificar trade-offs entre um monólito e a decomposição em serviços

## Requisitos e Restrições

- Assuma 10M de produtos, 5M de usuários ativos diários, ~50k leituras de catálogo/s no pico, ~2k checkouts/s no pico.
- A navegação deve ficar abaixo de ~200 ms no p99; um preço ou estoque desatualizado por alguns segundos é aceitável.
- O checkout nunca pode vender além do estoque: o estoque comprometido de um item não pode exceder o físico.
- A captura do pagamento e a criação do pedido devem ser atômicas na perspectiva do usuário.
- Estime o armazenamento do catálogo e de um ano de histórico de pedidos.

## Abordagem Sugerida

1. Desenhe o caminho da requisição para navegação vs. checkout e anote onde a consistência difere.
2. Dimensione o armazenamento do catálogo e um cache de leitura; calcule a taxa de acerto de cache necessária para atingir o QPS.
3. Projete a reserva de estoque: reservar-ao-adicionar-ao-carrinho com TTL, ou reservar-no-checkout.
4. Modele o checkout como uma saga (reservar → pagar → confirmar) com ações compensatórias em caso de falha.
5. Escolha chaves de particionamento para catálogo, carrinho e pedidos e defenda-as.

## Esboço de Arquitetura

```text
Clientes -> CDN/Edge -> API Gateway -> [svc Catálogo] -> BD Catálogo (réplicas) + cache Redis
                                     -> [svc Carrinho] -> Store Carrinho (Redis/Dynamo, por usuário)
                                     -> [svc Pedido]   -> BD Pedidos (shard por order_id)
                                          |-> svc Estoque -> BD Estoque (lock de linha / CAS)
                                          |-> svc Pagamento -> PSP externo
                          Eventos -> Kafka -> [indexador de busca, analytics, e-mail]

POST /cart/{userId}/items      { sku, qty } -> 200 carrinho
POST /checkout                 { userId }   -> 202 { orderId, status: PENDING }
GET  /orders/{orderId}                      -> 200 { status, items, total }

Product   { sku, title, price, attrs{}, categoryId }      // particiona por hash do sku
Inventory { sku, available, reserved, version }           // concorrência otimista
Order     { orderId, userId, items[], total, status, ts } // shard por orderId
```

## Tópicos de Aprofundamento

- **Consistência de estoque:** TTLs de reserva, lock otimista vs. pessimista, recuperação de venda excedente.
- **Saga de checkout:** orquestração vs. coreografia; chaves de idempotência no pagamento.
- **Trade-off 1 — monólito vs. serviços:** mais rápido de entregar vs. escalar leituras de catálogo de forma independente. Justifique separar o catálogo (muitas leituras) dos pedidos (muitas escritas).
- **Trade-off 2 — frescor do cache:** cache agressivo reduz carga no BD mas arrisca vender a um preço desatualizado; use TTLs curtos mais invalidação por evento.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo o diagrama acima.
- [ ] Estimativas de capacidade: QPS de catálogo, QPS de checkout, taxa de acerto de cache, armazenamento de catálogo + pedidos.
- [ ] Um plano de particionamento para catálogo, carrinho e pedidos com justificativa.
- [ ] Uma estratégia de cache com abordagem de invalidação.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Aplicar um único modelo de consistência em tudo — navegação e checkout têm necessidades diferentes.
- Reservar estoque sem TTL, fazendo carrinhos abandonados vazarem estoque para sempre.
- Tornar o pagamento não idempotente, fazendo um checkout repetido cobrar em dobro.
- Esquecer de dimensionar o cache; uma taxa de acerto baixa empurra silenciosamente toda a carga para o BD.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — estimativa de capacidade e padrões de escala.
- [Amazon Builders' Library: Using load shedding to avoid overload](https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/) — proteger o checkout sob carga.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — consistência e particionamento.
- [Padrão Saga (Microservices.io)](https://microservices.io/patterns/data/saga.html) — transações de checkout distribuídas.
