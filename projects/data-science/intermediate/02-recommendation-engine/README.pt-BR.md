# Motor de Recomendação (Filtragem Colaborativa)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Dada uma tabela esparsa de quem avaliou (ou comprou, ou clicou) o quê, preveja o que um usuário vai querer a seguir. Este projeto constrói um recomendador por filtragem colaborativa e — igualmente importante — uma forma honesta de medi-lo. A armadilha em recomendadores é que um número ingênuo de acurácia parece ótimo enquanto as recomendações são inúteis (sempre sugira o item mais popular e você vai "acertar" bastante). Você vai construir modelos de vizinhança e de fatoração de matriz, dividir as avaliações para que o conjunto de teste simule o futuro e avaliar com métricas de ranking que recompensam colocar os itens certos no topo.

## Pré-requisitos

- Conforto com NumPy/pandas e álgebra linear básica (produtos escalares, matrizes)
- Entender dados esparsos e por que uma matriz usuário-item completa é quase toda vazia
- Familiaridade com conceitos de divisão treino/teste
- Um conjunto de avaliações (MovieLens é a escolha canônica)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir uma matriz usuário-item e raciocinar sobre sua esparsidade
- Implementar filtragem colaborativa baseada em itens e em usuários com uma métrica de similaridade
- Aplicar fatoração de matriz (fatores latentes estilo SVD) e interpretar os fatores
- Dividir interações *temporalmente ou por leave-one-out* para que a avaliação imite a predição real
- Avaliar com métricas de ranking (Precision@K, Recall@K, NDCG) em vez de apenas RMSE

## Requisitos Funcionais

1. O sistema deve construir uma matriz de interação usuário-item a partir de eventos/avaliações brutos.
2. Deve produzir uma lista de recomendações top-N para qualquer usuário dado.
3. Deve implementar pelo menos duas abordagens (ex.: CF baseada em itens e fatoração de matriz).
4. As interações devem ser divididas em treino/validação/teste para que nenhuma interação de teste vaze para o treino.
5. O sistema deve reportar Precision@K, Recall@K e uma métrica sensível ao rank (NDCG ou MAP).
6. Deve tratar o caso cold-start: um usuário ou item sem histórico retorna um fallback sensato.
7. Deve comparar seus modelos com um baseline de popularidade e reportar o ganho.

## Marcos Sugeridos

1. **Marco 1 — Matriz e baseline:** Construa a matriz de interação e um baseline de mais populares.
2. **Marco 2 — CF e fatoração:** Adicione CF baseada em itens e um modelo estilo SVD.
3. **Marco 3 — Avaliar e ranquear:** Divida os dados, calcule métricas de ranking, compare ao baseline.

## Esboço de Dados e Interface

```text
Registro de interação
  user_id : string
  item_id : string
  rating  : float | implícito 1.0
  ts      : segundos epoch

Matriz R  (usuários x itens), quase toda vazia (esparsa)

Passos do pipeline
  1. dividir: leave-last-N-out por usuário -> treino / valid / teste
  2. construir R apenas com o treino
  3. modelar:
       item-CF  -> similaridade(item_i, item_j) via cosseno
       MF       -> R ~= U * V^T  (fatores latentes)
  4. recomendar: pontuar itens não vistos, pegar top-N
  5. avaliar no teste: Precision@K, Recall@K, NDCG@K
  6. comparar vs baseline de popularidade -> ganho
```

## Desafios Extras

- Adicione um modelo híbrido misturando features de conteúdo (gênero, tags) com o sinal colaborativo.
- Introduza diversidade/novidade no ranking para não trazer só os campeões de bilheteria.
- Suporte feedback implícito com ponderação por confiança em vez de avaliações explícitas.
- Sirva recomendações atrás de uma pequena API que retorne top-N para um id de usuário.

## Definição de Pronto

- [ ] Nenhuma interação de teste jamais fica visível a um modelo durante o treino.
- [ ] Listas top-N são produzidas para usuários, incluindo fallbacks de cold-start.
- [ ] Pelo menos dois modelos são avaliados com as mesmas métricas de ranking.
- [ ] Um baseline de popularidade é incluído e superado (ou a diferença é explicada).
- [ ] As definições das métricas (valor de K, estratégia de divisão) estão documentadas.

## Armadilhas Comuns

- Dividir avaliações aleatoriamente, deixando o futuro de um usuário vazar para suas linhas de treino.
- Reportar só RMSE — um erro baixo ainda pode ranquear itens mal para o top-N.
- Esquecer o baseline de popularidade, sem saber se o modelo agrega valor.
- Tratar itens não vistos como não gostados (0) quando o feedback é implícito, não negativo.

## Recursos

- [Datasets MovieLens](https://grouplens.org/datasets/movielens/) — o benchmark padrão de recomendadores.
- [Google: curso de Sistemas de Recomendação](https://developers.google.com/machine-learning/recommendation) — CF e fatoração de matriz explicados.
- [Documentação da biblioteca Surprise](https://surprise.readthedocs.io/en/stable/) — referência para CF e divisões de avaliação.
- [Wikipedia: Discounted cumulative gain](https://en.wikipedia.org/wiki/Discounted_cumulative_gain) — como o NDCG recompensa a ordem do ranking.
