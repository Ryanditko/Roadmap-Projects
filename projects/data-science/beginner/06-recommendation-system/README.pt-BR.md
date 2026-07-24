# Sistema de Recomendação Simples

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

"Porque você assistiu X, talvez goste de Y" é movido por uma ideia surpreendentemente acessível: encontre coisas parecidas e recomende-as. Neste projeto você constrói um recomendador básico a partir de um conjunto de dados de avaliações usando métricas de similaridade — item-a-item ("quem gostou disto também gostou de...") ou usuário-a-usuário ("pessoas como você curtiram..."). Você vai construir a matriz de interação, calcular similaridades, gerar recomendações top-N e enfrentar os dois problemas de todo recomendador: o que fazer com usuários ou itens novos (cold start) e como saber se as recomendações prestam.

## Pré-requisitos

- Python básico e pandas
- NumPy e, idealmente, scikit-learn para funções de similaridade
- Entendimento de uma matriz como linhas-por-colunas de números
- Um conjunto de avaliações usuário-item — o [dataset MovieLens 100K](https://grouplens.org/datasets/movielens/100k/) é a escolha padrão

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir uma matriz de interação (avaliações) usuário-item a partir de registros brutos
- Calcular similaridade entre itens ou usuários com similaridade de cosseno
- Gerar recomendações top-N a partir dos escores de similaridade
- Raciocinar sobre o problema de cold start e oferecer um fallback por popularidade
- Avaliar recomendações com um conjunto separado usando precision@k ou recall@k

## Requisitos Funcionais

1. O sistema deve construir uma matriz usuário-item a partir de um arquivo de avaliações.
2. Deve calcular um escore de similaridade entre itens (ou usuários) usando uma métrica declarada.
3. Dado um usuário ou item, deve retornar as top-N recomendações mais relevantes.
4. Deve excluir das recomendações os itens que o usuário já avaliou.
5. Deve prover um fallback baseado em popularidade para usuários ou itens sem histórico.
6. Deve avaliar a qualidade em um conjunto de teste separado com precision@k ou recall@k.
7. Deve anexar uma razão curta a cada recomendação ("parecido com X que você avaliou alto").

## Marcos Sugeridos

1. **Marco 1 — Matriz:** Carregue avaliações e construa a matriz usuário-item, notando o quão esparsa ela é.
2. **Marco 2 — Recomendar:** Calcule similaridades e produza recomendações top-N, excluindo itens já vistos.
3. **Marco 3 — Avaliar e fallback:** Separe avaliações, meça precision@k e adicione um fallback de cold start.

## Esboço de Dados e Interface

```text
Matriz de interação (usuários x itens, majoritariamente vazia)
              item_1  item_2  item_3  ...  item_m
  user_1        5       -       3           -
  user_2        -       4       -           2
  ...          (esparsa: a maioria das células sem avaliação)

Requisição/resposta de recomendação
  recommend(user_id, n=5)
    -> [ { item_id, score, reason: "parecido com <item> que você avaliou 5" }, ... ]
       exclui itens já avaliados por user_id
    -> se user_id desconhecido: retorna os top-n itens mais populares

Similaridade
  cosseno(a, b) = dot(a, b) / (||a|| * ||b||)   em {0..1} para avaliações não-negativas
Avaliação
  precision@k = (itens relevantes no top-k) / k   nas avaliações separadas
```

## Desafios Extras

- Combine similaridade colaborativa com um prior de popularidade em um híbrido simples.
- Adicione diversidade para que o top-N não seja cinco itens quase idênticos.
- Compare recomendações baseadas em item vs em usuário no mesmo conjunto de teste.
- Adicione fatoração de matriz (SVD) e compare seu precision@k com a abordagem de similaridade.

## Definição de Pronto

- [ ] A matriz usuário-item é construída e sua esparsidade é relatada.
- [ ] As recomendações excluem itens que o usuário já avaliou.
- [ ] Um usuário em cold start recebe recomendações sensatas baseadas em popularidade.
- [ ] Precision@k ou recall@k é medido em um conjunto separado, não nos dados de treino.
- [ ] Cada recomendação carrega uma razão legível por humanos.

## Armadilhas Comuns

- Recomendar itens que o usuário já avaliou por esquecer de mascará-los.
- Ignorar a esparsidade — a maioria dos usuários avalia quase nada, então similaridade ingênua é ruidosa.
- Avaliar nas mesmas avaliações usadas para construir a matriz, inflando a pontuação.
- Deixar alguns itens campeões dominarem toda lista de recomendação.

## Recursos

- [MovieLens datasets (GroupLens)](https://grouplens.org/datasets/movielens/) — os dados clássicos de avaliações.
- [scikit-learn: Pairwise metrics (cosine similarity)](https://scikit-learn.org/stable/modules/metrics.html#cosine-similarity) — calculando similaridade de forma eficiente.
- [Google: Recommendation Systems crash course](https://developers.google.com/machine-learning/recommendation) — filtragem colaborativa e cold start explicados.
- [Wikipedia: Collaborative filtering](https://en.wikipedia.org/wiki/Collaborative_filtering) — os conceitos por trás dos métodos item- e usuário-baseados.
