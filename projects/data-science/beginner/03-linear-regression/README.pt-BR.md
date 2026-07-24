# Modelo de Regressão Linear

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Regressão linear é o "hello world" da modelagem preditiva, e vale a pena fazê-la direito porque todo hábito que você forma aqui — dividir os dados, medir o erro honestamente, checar as premissas — se propaga para todo modelo que você venha a construir. Neste projeto você prevê um alvo contínuo (preço de imóvel, eficiência de combustível, valor de gorjeta) a partir de um punhado de features, e então interpreta o que o modelo aprendeu e o quanto confiar nele. O objetivo não é uma pontuação alta; é entender por que a pontuação é o que é e o que os coeficientes de fato significam.

## Pré-requisitos

- Python básico e pandas
- scikit-learn instalado
- Conforto com a ideia de features (entradas) e um alvo (saída)
- Um conjunto de dados de regressão com alvo numérico — o [dataset California Housing do scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.fetch_california_housing.html) ou o [dataset Auto MPG da UCI](https://archive.ics.uci.edu/dataset/9/auto+mpg) funcionam bem

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Dividir os dados em conjuntos de treino e teste e explicar por que a divisão importa
- Treinar um modelo de regressão linear e gerar previsões
- Avaliar com R², RMSE e MAE, e explicar o que cada um mede
- Ler um gráfico de resíduos para checar se um modelo linear é apropriado
- Interpretar coeficientes nas unidades do problema, atento à escala das features

## Requisitos Funcionais

1. O fluxo deve carregar o conjunto de dados e selecionar um alvo numérico mais ao menos três features.
2. Deve dividir os dados em conjuntos de treino e teste antes de qualquer modelo ser ajustado.
3. Deve treinar um modelo de regressão linear apenas no conjunto de treino.
4. Deve relatar R², RMSE e MAE calculados no conjunto de teste separado.
5. Deve produzir um gráfico de resíduos (previsto vs resíduos) e comentar o padrão.
6. Deve relatar e interpretar os coeficientes aprendidos.
7. Deve usar validação cruzada k-fold para checar que a pontuação é estável, não uma divisão de sorte.

## Marcos Sugeridos

1. **Marco 1 — Preparar e dividir:** Carregue os dados, escolha alvo e features, escale se necessário, divida treino/teste.
2. **Marco 2 — Treinar e avaliar:** Ajuste o modelo, preveja no conjunto de teste, relate R²/RMSE/MAE.
3. **Marco 3 — Diagnosticar:** Plote resíduos, interprete coeficientes, rode validação cruzada para estabilidade.

## Esboço de Dados e Interface

```text
Entrada/Saída do modelo
  X (features):  matriz [n_amostras, n_features]  (numérico)
  y (alvo):      vetor [n_amostras]               (contínuo)
  previsão:      y_hat = intercepto + soma(coef_i * x_i)

Relatório de avaliação
  R2:    fração da variância explicada (1.0 = perfeito, 0 = baseline da média)
  RMSE:  erro nas unidades do alvo, penaliza erros grandes
  MAE:   erro absoluto médio nas unidades do alvo
  CV:    média +/- desvio do R2 ao longo de k folds

Checagem de resíduos
  plot(previsto, real - previsto)
    nuvem aleatória em torno de 0  -> modelo linear é razoável
    curva ou formato de funil       -> não-linearidade ou heterocedasticidade
```

## Desafios Extras

- Adicione features polinomiais e compare com o modelo linear puro sem sobreajustar.
- Aplique regularização Ridge ou Lasso e observe o efeito nos coeficientes.
- Crie uma nova feature a partir de colunas existentes e meça se ela ajuda.
- Adicione intervalos de previsão para que cada previsão carregue uma faixa de incerteza.

## Definição de Pronto

- [ ] O modelo é treinado apenas com dados de treino e pontuado com dados de teste não vistos.
- [ ] R², RMSE e MAE são todos relatados e explicados em uma frase cada.
- [ ] Existe um gráfico de resíduos e seu formato é interpretado.
- [ ] Coeficientes são relatados nas unidades do problema, com a escala considerada.
- [ ] A validação cruzada confirma que a pontuação de teste não é acaso de uma única divisão.

## Armadilhas Comuns

- Avaliar no conjunto de treino e comemorar uma pontuação que o modelo nunca repetirá.
- Interpretar coeficientes crus quando as features estão em escalas muito diferentes.
- Ajustar o scaler no conjunto de dados completo (vazando info de teste) em vez de apenas no treino.
- Perseguir um R² maior enquanto ignora resíduos que claramente mostram uma relação não-linear.

## Recursos

- [scikit-learn: Linear Models](https://scikit-learn.org/stable/modules/linear_model.html) — a referência para `LinearRegression`, Ridge e Lasso.
- [scikit-learn: Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html) — como e por que usar k-fold.
- [Wikipedia: Coefficient of determination (R²)](https://en.wikipedia.org/wiki/Coefficient_of_determination) — o que o R² diz e o que não diz.
- [STAT 501: Regression Methods (Penn State)](https://online.stat.psu.edu/stat501/) — um curso gratuito e completo sobre premissas de regressão.
