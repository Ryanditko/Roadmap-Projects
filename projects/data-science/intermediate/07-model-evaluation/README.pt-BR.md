# Framework de Avaliação de Modelos

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Dois modelos, um conjunto de dados — qual é melhor? A resposta honesta é mais difícil do que ler um único número de acurácia. Este projeto constrói um framework que avalia e compara modelos como um praticante cuidadoso faz: com validação cruzada apropriada, um conjunto de métricas adequadas à tarefa, intervalos de confiança em torno de cada score e um teste estatístico para decidir se um modelo realmente vence outro ou só teve um split mais sortudo. Você vai aprender que o *protocolo de avaliação* é em si uma decisão de projeto — a estratégia de split errada ou a métrica errada pode coroar o modelo errado com total confiança.

## Pré-requisitos

- Conforto para treinar classificadores/regressores e ler uma matriz de confusão
- Entender overfitting e a razão da divisão treino/validação/teste
- Familiaridade com a interface de estimadores do scikit-learn (fit/predict)
- Um conjunto rotulado (classificação ou regressão) para avaliar modelos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar validação cruzada k-fold, estratificada e de séries temporais e saber quando cada uma se aplica
- Montar um conjunto de métricas casado com a tarefa (ROC-AUC, PR-AUC, F1 para classificação; RMSE, MAE, R² para regressão)
- Anexar intervalos de confiança às métricas via folds de validação cruzada ou bootstrapping
- Rodar um teste estatístico (pareado, ex.: t-test reamostrado corrigido) para comparar dois modelos
- Diagnosticar over/underfitting com curvas de aprendizado e de validação

## Requisitos Funcionais

1. O framework deve suportar múltiplas estratégias de validação cruzada selecionáveis por execução.
2. Deve calcular um conjunto configurável de métricas apropriadas ao tipo de tarefa.
3. Deve reportar uma média e um intervalo de confiança (ou desvio-padrão entre folds) para cada métrica.
4. Deve comparar pelo menos dois modelos nos mesmos folds e reportar qual vence.
5. Deve aplicar um teste de significância estatística à comparação de modelos.
6. Deve produzir curvas de aprendizado e/ou de validação para diagnóstico de overfitting.
7. Deve manter o conjunto de teste final intocado até a última avaliação única.

## Marcos Sugeridos

1. **Marco 1 — CV e métricas:** Implemente as estratégias de CV e o conjunto de métricas.
2. **Marco 2 — Intervalos e curvas:** Adicione intervalos de confiança e curvas de aprendizado.
3. **Marco 3 — Comparar:** Rode os modelos nos folds compartilhados e teste a significância da diferença.

## Esboço de Dados e Interface

```text
Config de avaliação
  task    : "classification" | "regression"
  cv      : "kfold" | "stratified" | "timeseries"
  folds   : int
  metrics : [ ... apropriadas à tarefa ... ]
  models  : [ estimadorA, estimadorB ]

Passos
  1. reservar conjunto de TESTE final, intocado
  2. para cada modelo:
       cross_validate sobre os MESMOS folds
       coletar scores por fold
  3. resumir: média +/- IC por métrica
  4. comparar: teste pareado nos scores de fold -> p
  5. learning_curve(modelo) -> gap treino vs valid
  6. uma avaliação final no TESTE para o modelo escolhido

Saída: tabela de métricas + vencedor + significância
```

## Desafios Extras

- Adicione validação cruzada aninhada para que a busca de hiperparâmetros não vaze para o score.
- Suporte curvas de calibração e Brier score para classificadores probabilísticos.
- Adicione uma métrica sensível a custo guiada por uma matriz de custo fornecida pelo usuário.
- Gere um relatório de uma página em markdown/HTML por execução de comparação.

## Definição de Pronto

- [ ] Pelo menos três estratégias de CV são implementadas e escolhidas apropriadamente por tarefa.
- [ ] Toda métrica é reportada com uma média e um intervalo, nunca uma estimativa pontual solta.
- [ ] Dois modelos são comparados em folds idênticos com um teste de significância na diferença.
- [ ] Curvas de aprendizado/validação revelam over- ou underfitting.
- [ ] O conjunto de teste é avaliado exatamente uma vez, no final.

## Armadilhas Comuns

- Usar k-fold simples em dados desbalanceados em vez de folds estratificados.
- Usar CV aleatória em séries temporais, deixando o modelo espiar o futuro.
- Declarar um vencedor a partir de um gap de 0,2% que está bem dentro da variância entre folds.
- Ajustar hiperparâmetros no conjunto de teste, tornando o número final otimista.

## Recursos

- [scikit-learn: Validação cruzada](https://scikit-learn.org/stable/modules/cross_validation.html) — estratégias e armadilhas.
- [scikit-learn: Métricas e scoring](https://scikit-learn.org/stable/modules/model_evaluation.html) — o catálogo completo de métricas.
- [Nadeau & Bengio: Inference for the Generalization Error](https://link.springer.com/article/10.1023/A:1024068626366) — o t-test reamostrado corrigido.
- [scikit-learn: Curvas de aprendizado](https://scikit-learn.org/stable/modules/learning_curve.html) — diagnosticando viés vs variância.
