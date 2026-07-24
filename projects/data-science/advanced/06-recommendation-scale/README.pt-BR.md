# Sistema de Recomendação em Escala

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete um sistema de recomendação que permanece rápido e relevante quando o catálogo é enorme e o tráfego é pesado. A abordagem ingênua de "pontuar cada item para cada usuário" morre em escala, então este projeto se centra no padrão de dois estágios da indústria: um passo barato de recuperação de candidatos que reduz milhões de itens para algumas centenas, seguido de um modelo de ranking mais pesado sobre essa lista curta. Ao redor disso você vai tratar cold start, manter os resultados diversos, rodar atualizações online e medir qualidade com uma avaliação offline estilo A/B. O objetivo é um sistema cuja latência e qualidade ambas se sustentam conforme o catálogo cresce.

## Pré-requisitos

- Entendimento de filtragem colaborativa e fatoração de matrizes
- Experiência com embeddings e busca por vizinhos mais próximos aproximada
- Familiaridade com uma ferramenta de dados em streaming ou batch (Kafka, Spark ou similar)
- Conforto com métricas de ranking offline (recall@k, NDCG, MAP)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir uma arquitetura de recomendação de dois estágios: recuperação e depois ranking
- Usar embeddings mais indexação ANN para recuperação de candidatos sublinear
- Tratar cold start para novos usuários e itens
- Equilibrar relevância contra diversidade e frescor no ranker
- Avaliar recomendações offline com métricas de ranking e raciocinar sobre o design de A/B online

## Requisitos Funcionais

1. O sistema deve gerar candidatos para um usuário a partir de um catálogo grande em tempo sublinear (ANN, não varredura completa).
2. Um modelo de ranking deve repontuar candidatos usando features mais ricas que o estágio de recuperação.
3. O sistema deve tratar cold start para novos usuários e novos itens com uma estratégia explícita.
4. As recomendações devem incluir um passo de diversidade/de-duplicação para que os resultados não sejam quase idênticos.
5. O sistema deve suportar atualizações online para que novas interações influenciem recomendações futuras.
6. A avaliação offline deve reportar métricas de ranking (recall@k, NDCG) em um período held-out.
7. O sistema deve expor uma API de recomendação retornando itens ranqueados com scores.

## Requisitos Não Funcionais

- **Latência:** p95 da recomendação ponta a ponta dentro de um orçamento declarado no QPS alvo.
- **Escalabilidade:** a latência de recuperação deve permanecer sublinear conforme o catálogo cresce uma ordem de magnitude.
- **Frescor:** novas interações devem influenciar as recomendações dentro de um atraso limitado.
- **Consistência:** o split de avaliação offline deve respeitar o tempo (sem vazamento do futuro para o passado).

## Marcos Sugeridos

1. **Marco 1 — Embeddings e recuperação:** Treine embeddings de item/usuário e construa um índice ANN para recuperação de candidatos.
2. **Marco 2 — Ranking:** Adicione um modelo de ranking sobre os candidatos com features de interação e contexto.
3. **Marco 3 — Cold start e diversidade:** Adicione tratamento de cold start e um passo de re-ranking por diversidade.
4. **Marco 4 — Online e avaliação:** Adicione atualizações online e um harness de avaliação offline que respeita o tempo.

## Esboço de Dados e Interface

```text
 interações (user, item, ts, sinal)
        |
        v
 +------------------+   treina embeddings user/item
 | Modelo embedding |
 +--------+---------+
          v
 +------------------+   milhões -> ~centenas
 | Recuperação ANN  |   (FAISS/ScaNN)  <-- fallback cold start: popularidade/conteúdo
 +--------+---------+
          v
 +------------------+   features ricas: recência, contexto, cross features
 | Modelo de ranking|
 +--------+---------+
          v
 +------------------+   MMR / limites por categoria
 | Re-rank divers.  |
 +--------+---------+
          v
   GET /recommend?user_id=U&k=20 -> [ {item_id, score, reason} ... ]

 Aval: split por TEMPO; recall@k, NDCG@k na janela futura
 Online: stream de novas interações -> atualiza embeddings/features
```

## Desafios Extras

- Adicione um bandit (epsilon-greedy ou Thompson sampling) para exploração vs explotação.
- Suporte recomendações baseadas em sessão usando modelos de sequência.
- Adicione uma feature store compartilhada entre treino offline e serving online.
- Pré-calcule recomendações para os usuários mais quentes e sirva-as de um cache.

## Definição de Pronto

- [ ] A recuperação de candidatos usa ANN e permanece sublinear conforme o catálogo cresce.
- [ ] Um estágio de ranking repontua candidatos com features além das usadas na recuperação.
- [ ] Usuários e itens em cold start recebem recomendações sensatas via um fallback explícito.
- [ ] Um passo de diversidade evita listas de resultados quase duplicadas.
- [ ] A avaliação offline respeita a ordenação temporal e reporta recall@k e NDCG.

## Armadilhas Comuns

- Pontuar o catálogo inteiro por requisição e estourar o orçamento de latência conforme ele cresce.
- Avaliar com um split aleatório, vazando interações futuras para a janela de treino.
- Otimizar relevância pura até que toda lista pareça idêntica e os usuários se desengajem.
- Ignorar cold start, de modo que novos itens nunca são exibidos e nunca reúnem sinal.

## Recursos

- [Google: curso de Sistemas de Recomendação](https://developers.google.com/machine-learning/recommendation) — recuperação, ranking e geração de candidatos.
- [Documentação do FAISS](https://faiss.ai/) — busca por vizinhos mais próximos aproximada em escala.
- [Deep Neural Networks for YouTube Recommendations (Covington et al., 2016)](https://research.google/pubs/pub45530/) — a arquitetura canônica de dois estágios.
- [Matrix Factorization Techniques for Recommender Systems (Koren et al., 2009)](https://ieeexplore.ieee.org/document/5197422) — filtragem colaborativa fundamental.
