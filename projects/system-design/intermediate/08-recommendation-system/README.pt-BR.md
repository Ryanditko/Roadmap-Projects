# Projete um Sistema de Recomendação

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de um sistema de recomendação como os que ficam por trás das fileiras da Netflix ou de uma faixa "você também pode gostar" de e-commerce: dado um usuário, produza uma lista curta e ranqueada de itens com os quais ele provavelmente vai interagir, rápido o bastante para renderizar no carregamento da página. O padrão arquitetural é um funil de dois estágios — geração barata de candidatos sobre milhões de itens, depois ranqueamento caro sobre algumas centenas. Os problemas difíceis são o cold-start de usuário, a latência de serving e manter as recomendações frescas. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento de filtragem colaborativa vs. baseada em conteúdo em nível conceitual
- Familiaridade com sistemas offline (batch) vs. online (serving)
- Noção de embeddings e busca aproximada de vizinhos mais próximos (ANN)
- Conforto para estimar QPS de serving e armazenamento de dados pré-computados

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma arquitetura de dois estágios: geração de candidatos → ranqueamento
- Estimar QPS de serving e armazenamento de recomendações pré-computadas e embeddings
- Tratar o problema de cold-start para novos usuários e novos itens
- Projetar um cache de serving para que as recomendações renderizem dentro do orçamento de latência
- Justificar trade-offs entre recomendações pré-computadas (batch) e em tempo real

## Requisitos e Restrições

- Assuma 50M de usuários, 10M de itens, ~30k requisições de recomendação/s no pico.
- Uma resposta de recomendação deve retornar em menos de ~150 ms no p99.
- Novos usuários (sem histórico) e novos itens (sem interações) ainda devem receber resultados razoáveis.
- As recomendações devem refletir o comportamento recente em minutos, não dias.
- Estime o QPS de serving, o armazenamento de embeddings e o armazenamento de recs pré-computadas.

## Abordagem Sugerida

1. Separe o offline (treinar modelos, construir embeddings, pré-computar em batch) do serving online.
2. Projete a geração de candidatos: ANN sobre embeddings de itens, mais fontes de popularidade e recência.
3. Projete o ranqueamento: um modelo leve pontuando algumas centenas de candidatos por requisição.
4. Trate o cold start com fallbacks de popularidade e baseados em conteúdo.
5. Adicione um cache de serving chaveado por usuário com TTL curto para frescor.

## Esboço de Arquitetura

```text
Offline: interações -> [Treinamento] -> embeddings de item/usuário + modelo de ranqueamento
                        -> pré-computa top-N por usuário em batch -> Store de Recs

Online:  requisição(userId) -> [svc Serving]
             1. candidatos: ANN(emb usuário) UNION populares UNION recentes  (centenas)
             2. ranqueia: modelo.score(usuário, candidato) -> top-K
             3. filtra: já-vistos, regras de negócio
          -> cache(userId, TTL) -> retorna

GET /recommendations?userId&context -> 200 { items[], modelVersion }
POST /events { userId, itemId, action, ts } -> 204   // alimenta sinais frescos

UserEmb { userId -> vetor[d] }              // particiona por userId
ItemEmb { itemId -> vetor[d] }              // índice ANN, com shards
Recs    { userId -> [itemId, score]... }    // pré-computado, cache com TTL
```

## Tópicos de Aprofundamento

- **Geração de candidatos:** busca ANN sobre embeddings; combinação de múltiplas fontes de candidatos.
- **Cold start:** features baseadas em conteúdo e popularidade para usuários/itens sem histórico.
- **Trade-off 1 — recs pré-computadas em batch vs. tempo real:** pré-computar top-N por usuário torna o serving uma busca em cache mas fica desatualizado entre execuções e desperdiça computação com usuários inativos; o rerank em tempo real é fresco mas crítico em latência. Justifique pré-computação + rerank em tempo real do topo.
- **Trade-off 2 — complexidade do modelo vs. latência:** um modelo de ranqueamento mais rico melhora a qualidade mas arrisca o orçamento de 150 ms; mantenha a geração de candidatos barata e gaste o orçamento ranqueando um conjunto pequeno.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo a arquitetura de dois estágios acima.
- [ ] Estimativas de capacidade: QPS de serving, armazenamento de embeddings, armazenamento de recs pré-computadas.
- [ ] Um plano de particionamento para embeddings e o store de recs.
- [ ] Uma estratégia de cache para recomendações servidas com TTL e política de frescor.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Ignorar o cold start, fazendo novos usuários receberem uma lista vazia ou sem sentido.
- Ranquear sobre o catálogo inteiro por requisição em vez de um conjunto de candidatos limitado.
- Pré-computar recs para cada usuário diariamente, desperdiçando computação com a maioria inativa.
- Nunca filtrar itens já vistos, fazendo as mesmas recomendações se repetirem para sempre.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — design de sistemas batch vs. serving.
- [Google ML: curso de Recommendation Systems](https://developers.google.com/machine-learning/recommendation) — geração de candidatos e ranqueamento.
- [Paper de recomendações do YouTube (RecSys 2016)](https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/) — o funil de dois estágios em produção.
- [Faiss: busca eficiente por similaridade](https://github.com/facebookresearch/faiss) — vizinhos mais próximos aproximados para candidatos.
