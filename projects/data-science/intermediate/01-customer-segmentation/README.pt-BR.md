# Segmentação de Clientes (Clustering)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Pegue uma tabela de comportamento de clientes — recência, frequência, valor monetário, tempo de casa, mix de produtos — e descubra os grupos naturais escondidos ali dentro. Este é um projeto não supervisionado, então não há rótulo para prever nem acurácia para perseguir; a vitória é um conjunto de segmentos que um time de marketing consiga de fato nomear e usar ("gastadores dormentes", "caçadores de ofertas recentes"). O trabalho real está antes do algoritmo: escalar features para que distância signifique algo, escolher quantos clusters manter e provar que o agrupamento é estável, não um artefato de uma única semente aleatória.

## Pré-requisitos

- Conforto com uma biblioteca de dataframe (pandas, Polars ou R) e gráficos básicos
- Familiaridade com estatística descritiva e padronização (z-score, min-max)
- Um primeiro contato com K-means ou raciocínio de vizinhos mais próximos
- Um conjunto de dados tabular de clientes (ex.: uma amostra de e-commerce ou RFM)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Criar features no estilo RFM e justificar cada escolha de escala que fizer
- Rodar K-means e escolher *k* usando o método do cotovelo **e** o silhouette score, não apenas um
- Comparar um método de centroide (K-means) com um de densidade (DBSCAN) e explicar quando cada um se encaixa
- Reduzir a dimensionalidade com PCA para visualização sem vazá-la para o clustering
- Perfilar cada cluster em uma persona legível pelo negócio, com estatísticas de apoio

## Requisitos Funcionais

1. O pipeline deve carregar registros brutos de clientes e derivar uma tabela de features documentada.
2. As features devem ser escaladas, e o scaler escolhido deve ser ajustado só nos dados de treino e reutilizado.
3. A ferramenta deve rodar pelo menos dois algoritmos de clustering e reportar os tamanhos dos clusters de cada um.
4. O número ótimo de clusters deve ser escolhido usando pelo menos dois diagnósticos independentes.
5. Cada cluster resultante deve ser resumido com médias por feature e uma persona escrita.
6. A estabilidade dos clusters deve ser verificada reexecutando com sementes ou subamostras diferentes.
7. Os resultados devem ser visualizados em 2D (PCA ou t-SNE) com clusters coloridos.

## Marcos Sugeridos

1. **Marco 1 — Tabela de features:** Construa e escale features RFM a partir de transações brutas.
2. **Marco 2 — Agrupar e ajustar:** Rode K-means, varra *k*, escolha-o com cotovelo + silhouette.
3. **Marco 3 — Comparar e perfilar:** Adicione DBSCAN, cheque estabilidade e escreva as personas.

## Esboço de Dados e Interface

```text
Tabela de features (uma linha por cliente)
  customer_id   : string
  recency_days  : inteiro   (dias desde a última compra)
  frequency     : inteiro   (nº de pedidos na janela)
  monetary      : float     (gasto total)
  tenure_days   : inteiro
  avg_basket    : float

Passos do pipeline
  1. agregar transações -> features por cliente
  2. escalar features (StandardScaler, ajustar no split de treino)
  3. agrupar (KMeans k=2..10 | varredura de eps do DBSCAN)
  4. pontuar (cotovelo da inércia, silhouette) -> escolher k
  5. projetar para 2D (PCA) apenas para o gráfico
  6. perfilar: groupby(cluster).mean() -> tabela de personas
```

## Desafios Extras

- Adicione Gaussian Mixture Models e compare atribuição suave vs rígida de clusters.
- Classifique novos clientes em segmentos existentes sem reajustar o modelo inteiro.
- Pondere features monetárias pela recência para capturar mudanças recentes de comportamento.
- Construa um pequeno dashboard que deixe um stakeholder filtrar clientes por segmento.

## Definição de Pronto

- [ ] A tabela de features é reproduzível a partir dos dados brutos com um passo documentado.
- [ ] O escalonamento é ajustado só no split de treino — sem vazamento do conjunto completo.
- [ ] O *k* escolhido é defendido com evidências de cotovelo e silhouette.
- [ ] Pelo menos dois algoritmos são comparados com uma recomendação clara.
- [ ] Todo cluster tem uma persona nomeada apoiada por estatísticas por feature.

## Armadilhas Comuns

- Agrupar features não escaladas, deixando `monetary` dominar toda distância.
- Ler o gráfico do cotovelo como verdade absoluta — ele costuma ser ambíguo; combine com silhouette.
- Tratar os pontos de ruído do DBSCAN (rótulo -1) como um cluster real nos perfis.
- Ajustar PCA ou o scaler em todos os dados e depois estranhar segmentos instáveis.

## Recursos

- [scikit-learn: Clustering](https://scikit-learn.org/stable/modules/clustering.html) — algoritmos e trade-offs lado a lado.
- [scikit-learn: Análise de silhouette](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html) — escolhendo *k* visualmente.
- [Wikipedia: RFM (market research)](https://en.wikipedia.org/wiki/RFM_(market_research)) — o framework clássico de features de clientes.
- [scikit-learn: PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) — redução de dimensionalidade para visualização.
