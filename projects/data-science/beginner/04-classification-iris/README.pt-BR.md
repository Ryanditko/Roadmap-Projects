# Modelo de Classificação (dataset Iris)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

O dataset Iris — 150 flores, quatro medidas, três espécies — é o clássico primeiro problema de classificação, e é pequeno o suficiente para você focar inteiramente no fluxo de trabalho em vez de brigar com os dados. Neste projeto você treina um modelo para prever a espécie de uma flor a partir das medidas de pétala e sépala, e então o avalia honestamente com uma matriz de confusão e métricas por classe. Como duas das três espécies se sobrepõem, você também aprende que só a acurácia pode esconder fraquezas reais, e que precisão e recall contam a história mais completa.

## Pré-requisitos

- Python básico e pandas
- scikit-learn instalado
- Entendimento da diferença entre classificação (categorias) e regressão (números)
- O conjunto de dados já vem embutido: `sklearn.datasets.load_iris` — veja a [referência Iris do scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Formular um problema de classificação multiclasse e preparar dados rotulados
- Treinar um classificador (Árvore de Decisão, k-NN ou Random Forest) e prever classes
- Ler uma matriz de confusão para ver quais classes se confundem
- Calcular e interpretar acurácia, precisão, recall e F1 por classe
- Usar validação cruzada e uma divisão treino/teste para estimar o desempenho real

## Requisitos Funcionais

1. O fluxo deve carregar os dados Iris e inspecionar o balanceamento de classes e as faixas das features.
2. Deve dividir os dados em conjuntos de treino e teste estratificados.
3. Deve treinar ao menos um classificador no conjunto de treino.
4. Deve relatar acurácia mais precisão, recall e F1 por classe no conjunto de teste.
5. Deve produzir e interpretar uma matriz de confusão.
6. Deve usar validação cruzada para confirmar que o resultado é estável.
7. Deve comparar ao menos dois classificadores diferentes na mesma divisão.

## Marcos Sugeridos

1. **Marco 1 — Explorar e dividir:** Carregue Iris, cheque o balanceamento de classes, faça uma divisão treino/teste estratificada.
2. **Marco 2 — Treinar e avaliar:** Ajuste um classificador, construa a matriz de confusão, relate as métricas de classificação.
3. **Marco 3 — Comparar:** Treine um segundo modelo, valide ambos por cruzamento e explique qual escolheria e por quê.

## Esboço de Dados e Interface

```text
Entrada/Saída do modelo
  features (X):  [sepal_length, sepal_width, petal_length, petal_width]  (cm, float)
  alvo (y):      espécie em {setosa, versicolor, virginica}
  previsão:      um dos três rótulos de espécie

Matriz de confusão (linhas = real, colunas = previsto)
                 setosa  versicolor  virginica
  setosa            50        0          0
  versicolor         0       47          3
  virginica          0        2         48
  -> setosa é trivialmente separável; a confusão vive entre versicolor & virginica

Relatório por classe
  precisão = TP / (TP + FP)   recall = TP / (TP + FN)   F1 = média harmônica
```

## Desafios Extras

- Ajuste hiperparâmetros com grid search e relate se realmente ajudou.
- Reduza a duas features e plote a fronteira de decisão para ver como o classificador divide o espaço.
- Adicione curvas ROC um-contra-todos e compare o AUC entre classes.
- Construa uma pequena interface de previsão que recebe quatro medidas e retorna uma espécie com confiança.

## Definição de Pronto

- [ ] A divisão treino/teste é estratificada para preservar o balanceamento de classes.
- [ ] Acurácia e precisão/recall/F1 por classe são todos relatados.
- [ ] A matriz de confusão é mostrada e suas células fora da diagonal são explicadas.
- [ ] Ao menos dois classificadores são comparados nos mesmos dados.
- [ ] A validação cruzada confirma que a pontuação do modelo escolhido é estável.

## Armadilhas Comuns

- Julgar o modelo só pela acurácia, sem perceber que uma classe é sistematicamente confundida.
- Fazer uma divisão não estratificada, deixando uma espécie sub-representada no conjunto de teste.
- Vazar informação escalando ou ajustando no conjunto completo antes da divisão.
- Sobreajustar uma Árvore de Decisão profunda a 150 pontos e confundir memorização com habilidade.

## Recursos

- [scikit-learn: Iris dataset reference](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html) — o loader embutido e a descrição das features.
- [scikit-learn: Classification metrics](https://scikit-learn.org/stable/modules/model_evaluation.html#classification-metrics) — precisão, recall, F1 e o relatório de classificação.
- [scikit-learn: Confusion matrix](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.confusion_matrix.html) — como computar e ler.
- [Google ML Crash Course: Classification](https://developers.google.com/machine-learning/crash-course/classification) — base acessível sobre as métricas.
