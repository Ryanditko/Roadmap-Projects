# Projete um Sistema de Caronas Compartilhadas

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de uma plataforma de caronas como Uber ou 99: passageiros solicitam corridas, o sistema encontra o motorista disponível mais próximo, ambos se veem se mover em tempo real, e o pagamento é liquidado ao fim da corrida. Os desafios centrais são armazenar e consultar milhões de pontos GPS em movimento, casar oferta e demanda em milissegundos e precificar dinamicamente quando a demanda dispara. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento de conceitos de indexação geoespacial (geohash, quadtree, S2/H3)
- Familiaridade com transporte em tempo real (WebSockets, conexões persistentes)
- Noção de sharding e problemas de partição quente
- Conforto para estimar throughput de escrita a partir de um fluxo de pings de localização

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher um índice geoespacial e explicar consultas de motorista mais próximo
- Estimar o QPS de escrita de atualizações de localização e o armazenamento de motoristas ativos
- Projetar um serviço de matching que evite atribuir um motorista em duplicidade
- Particionar dados de localização geograficamente sem criar células quentes
- Justificar trade-offs entre precisão do geohash e amplitude da consulta

## Requisitos e Restrições

- Assuma 1M de motoristas ativos, cada um enviando localização a cada 4 s; ~5M de passageiros/dia.
- Uma consulta de motorista mais próximo deve retornar em menos de ~100 ms dentro de uma cidade.
- Um motorista nunca pode ser casado com dois passageiros ao mesmo tempo.
- Dados de localização têm muitas escritas e são efêmeros; registros de corrida são duráveis.
- Estime o QPS de escrita de localização, o tamanho do índice geoespacial e o armazenamento de histórico de corridas/ano.

## Abordagem Sugerida

1. Calcule o QPS de escrita de localização (motoristas ÷ intervalo de ping) e dimensione o índice ao vivo.
2. Escolha uma codificação geoespacial (geohash/H3) e mapeie células para shards.
3. Projete o matching: consulte motoristas candidatos em células próximas e reserve um atomicamente.
4. Projete o canal em tempo real para atualizações de posição de motorista/passageiro.
5. Adicione preço dinâmico guiado pela razão oferta/demanda por célula.

## Esboço de Arquitetura

```text
App motorista -- ping localização (4s) --> [svc Localização] -> Índice geo (Redis GEO / H3 -> shard)
App passageiro -- solicitar corrida ----> [svc Matching] -> consulta células próximas -> reserva motorista (CAS)
                                          |-> [svc Corrida] -> BD Corridas (shard por tripId)
                                          |-> [svc Preço] -> surge = f(demanda/oferta por célula)
Posições em tempo real <-- gateway WebSocket --> ambos os apps

POST /rides            { riderId, pickup{lat,lng} } -> 202 { tripId, driverId, eta }
POST /location         { driverId, lat, lng, ts }   -> 204
GET  /trips/{tripId}                                -> 200 { status, route, fare }

Driver { driverId, cell, lat, lng, status, ts }   // chaveia por célula, TTL em pings antigos
Trip   { tripId, riderId, driverId, state, fare }  // shard por tripId
```

## Tópicos de Aprofundamento

- **Indexação geoespacial:** células geohash vs. H3; buscas de vizinhos nas bordas das células.
- **Integridade do matching:** reserva atômica, timeouts, re-matching quando o motorista recusa.
- **Trade-off 1 — precisão do geohash:** células finas dão proximidade precisa mas exigem consultar muitas células vizinhas; células grosseiras retornam candidatos demais. Justifique uma precisão ajustada à densidade da cidade.
- **Trade-off 2 — durabilidade do store de localização:** persistir cada ping é caro e raramente lido; mantenha posições ao vivo em memória com TTL e só persista as rotas das corridas.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo a arquitetura acima.
- [ ] Estimativas de capacidade: QPS de escrita de localização, memória do índice geo, armazenamento de histórico de corridas/ano.
- [ ] Um plano de particionamento geográfico que trate células quentes (ex.: centro no horário de pico).
- [ ] Uma estratégia de cache/memória para posições ao vivo com política de TTL.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Armazenar cada ping GPS de forma durável, explodindo o custo de escrita de dados que ninguém consulta.
- Ignorar casos de borda de célula, de modo que um motorista a um metro do limite nunca é casado.
- Matching não atômico que atribui um motorista a dois passageiros sob concorrência.
- Fazer sharding puramente por geografia, tornando um estádio ou aeroporto um shard quente permanente.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de sharding e tempo real.
- [Uber Engineering: índice hexagonal H3](https://www.uber.com/blog/h3/) — particionamento geoespacial em produção.
- [Comandos Geoespaciais do Redis](https://redis.io/docs/latest/develop/data-types/geospatial/) — consultas de proximidade com GEO.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — particionamento e pontos quentes.
