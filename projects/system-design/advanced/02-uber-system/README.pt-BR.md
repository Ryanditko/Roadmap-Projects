# Projete uma Plataforma de Caronas como a Uber

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete uma plataforma de transporte por aplicativo no estilo da Uber: passageiros solicitam uma viagem, o sistema encontra um motorista próximo disponível em segundos, rastreia ambas as partes em tempo real e liquida o pagamento quando a corrida termina. A parte difícil é o casamento de dois fluxos contínuos — uma mangueira de pings de GPS de motoristas e uma rajada de solicitações de passageiros — resolvido por um motor de matching geoespacial sob um orçamento apertado de latência, tudo enquanto o preço dinâmico reequilibra oferta e demanda. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Conforto com sistemas em tempo real (WebSockets, pub/sub, streaming)
- Familiaridade com conceitos de indexação geoespacial (geohash, quadtrees, células S2)
- Entendimento de consistência eventual e escritas idempotentes
- Noção básica de fluxos de pagamento e transações distribuídas

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um índice geoespacial que responde "motoristas disponíveis mais próximos" em escala de cidade
- Modelar a mangueira de localização de motoristas e escolher caminhos de escrita/leitura que sobrevivem ao volume
- Projetar um algoritmo de matching que equilibra o tempo de espera do passageiro com a utilização do motorista
- Raciocinar sobre preço dinâmico como um laço de realimentação de oferta/demanda
- Tratar pagamentos com semântica exactly-once apesar de retentativas e falhas

## Requisitos e Restrições

1. Casar um passageiro com um motorista em poucos segundos, p99 abaixo de ~2 s em áreas densas.
2. Ingerir atualizações de localização a cada ~4 s de milhões de motoristas ativos.
3. Garantir que um motorista seja oferecido a no máximo um passageiro por vez (sem despacho duplo).
4. Calcular tarifas deterministicamente e cobrar exatamente uma vez por viagem concluída.
5. Aplicar multiplicadores de surge por célula geográfica, atualizados em cadência curta.
6. Mirar 99,95% de disponibilidade; um match falho deve degradar graciosamente com retentativa.
7. Aplicar conformidade por região (regras de preço, verificação de motorista, residência de dados).

## Abordagem Sugerida

Estime primeiro a vazão de escrita de localização (motoristas × frequência de ping) — ela ofusca o tráfego de leitura e molda sua escolha de armazenamento. Particione o mundo em células geográficas (ex.: S2/geohash) e mantenha um índice quente em memória das posições de motoristas por célula, reconstruído a partir de um stream. Modele o matching como: localizar células candidatas → consultar motoristas próximos → ranquear por ETA → oferecer com um lock/TTL curto. Trate o surge como um serviço separado que lê demanda/oferta por célula e publica multiplicadores. Mantenha o ciclo de vida da viagem como uma máquina de estados apoiada em eventos duráveis, e isole pagamentos atrás de uma chave de idempotência para que retentativas nunca cobrem em dobro.

## Esboço de Arquitetura

```text
App Motorista ──ping──> Ingestão de localização (stream) ──> Índice geo (memória por célula)
                                                                  ^
App Passageiro ──solicita──> Svc Viagem ──match──> Svc Matching ─┘
                          │
                          ├──> Svc Preço/Surge ──> demanda/oferta por célula
                          └──> Svc Pagamentos ──(chave idempotência)──> ledger + PSP

Ciclo de vida da viagem (máquina de estados):
REQUESTED -> MATCHED -> ACCEPTED -> IN_PROGRESS -> COMPLETED -> PAID
                 └────────> TIMED_OUT/CANCELLED

APIs principais:
POST /trips                 -> { tripId, status: REQUESTED }
POST /drivers/{id}/location -> 202 (fire-and-forget, em lote)
POST /trips/{id}/accept     -> { status: ACCEPTED }  (lado do motorista)
GET  /trips/{id}            -> { status, driverLoc, etaSeconds }

Modelo de dados (esboço):
Driver{ id, cellId, status: AVAILABLE|BUSY, lastPingAt }
Trip{ id, riderId, driverId, state, fare, surgeMultiplier }
```

## Tópicos de Aprofundamento

- **Indexação geoespacial:** S2 vs geohash vs quadtree; trade-off entre tamanho da célula e fan-out da consulta.
- **Matching sob contenção:** locks de despacho, TTLs, evitar oferta dupla, backoff em recusa.
- **Laço de preço dinâmico:** medir demanda/oferta, amortecer oscilação, justiça e tetos.
- **Idempotência de pagamento:** cobrança exactly-once, saga/compensação para capturas falhas.
- **Mitigação de hotspot:** picos de saída de estádio concentrados em poucas células.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com a arquitetura e a máquina de estados da viagem, refinado.
- [ ] Estimativas de capacidade para escritas de localização, memória do índice geo e QPS de match no pico.
- [ ] O algoritmo de matching descrito com sua estratégia de consistência e locking.
- [ ] Uma análise de falha/DR: perda de nó do índice geo, queda do PSP de pagamento, isolamento de região.
- [ ] Plano de mitigação de hotspot para picos de demanda concentrados em áreas pequenas.

## Armadilhas Comuns

- Armazenar cada ping de GPS em um BD fortemente consistente — o volume de escrita vai esmagá-lo; use stream.
- Oferecer um motorista a múltiplos passageiros porque o despacho não tem lock/TTL.
- Cobrar duas vezes em retentativa porque o caminho de pagamento não tem chave de idempotência.
- Surge global em vez de por célula, fazendo o preço oscilar de forma injusta.
- Escolher células geográficas grandes demais (matches ruins) ou pequenas demais (fan-out enorme de consulta).

## Recursos

- [Uber Engineering Blog](https://www.uber.com/en/blog/engineering/) — relatos reais de matching e sistemas geo.
- [S2 Geometry](https://s2geometry.io/) — a biblioteca de células esféricas usada para indexação geo.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de sharding, cache e tempo real.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — consistência, streams e idempotência a fundo.
- [Stripe: Requisições idempotentes](https://docs.stripe.com/api/idempotent_requests) — o padrão canônico de pagamento exactly-once.
