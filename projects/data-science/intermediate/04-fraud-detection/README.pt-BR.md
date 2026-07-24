# Modelo de Detecção de Fraude

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Fraude é rara — talvez uma transação em mil — e essa raridade quebra toda intuição que você tem sobre acurácia. Um modelo que prevê "não é fraude" para tudo atinge 99,9% de acurácia e pega zero fraudes. Este projeto é sobre construir um classificador que de fato encontra a agulha: criar features que expõem padrões de fraude, tratar o desbalanceamento de classes de forma honesta e avaliar com métricas que sobrevivem ao desbalanço (precisão, recall, PR-AUC). Você também vai encarar a realidade de negócio de que um falso positivo (bloquear um cliente real) e um falso negativo (deixar passar fraude) custam valores bem diferentes, então o limiar de decisão é uma escolha de projeto, não um padrão.

## Pré-requisitos

- Conforto para treinar um classificador (regressão logística, ensembles de árvores) em scikit-learn ou similar
- Entender a matriz de confusão e precisão/recall
- Familiaridade com divisão treino/validação/teste
- Um conjunto de transações desbalanceado (ex.: o dataset de fraude de cartão do Kaggle)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Criar features de transação (velocidade, desvio de valor, hora do dia) que expõem fraude
- Tratar desbalanceamento com reamostragem (SMOTE) e/ou pesos de classe, aplicados só ao treino
- Escolher métricas de avaliação significativas sob forte desbalanço (PR-AUC, recall a precisão fixa)
- Ajustar um limiar de decisão contra um trade-off de custo explícito, não o padrão 0.5
- Produzir explicações por predição para apoiar um analista revisando um alerta

## Requisitos Funcionais

1. O pipeline deve dividir os dados em treino/validação/teste *antes* de qualquer reamostragem ou ajuste.
2. Reamostragem ou ponderação de classe deve ser aplicada só ao fold de treino.
3. Deve criar pelo menos três features relevantes para fraude a partir dos campos brutos.
4. Deve reportar precisão, recall, F1 e PR-AUC — não acurácia sozinha.
5. Deve expor um limiar de decisão ajustável e mostrar a curva de trade-off precisão/recall.
6. Deve produzir um score de fraude por transação, não apenas um rótulo rígido.
7. Deve incluir uma forma de explicar por que uma transação foi sinalizada.

## Marcos Sugeridos

1. **Marco 1 — Dividir e features:** Divida os dados e então crie features de padrão de fraude.
2. **Marco 2 — Treinar e balancear:** Treine modelos com tratamento de desbalanço no fold de treino.
3. **Marco 3 — Avaliar e limiar:** Reporte métricas sensíveis ao desbalanço e ajuste o limiar.

## Esboço de Dados e Interface

```text
Registro de transação
  txn_id     : string
  amount     : float
  ts         : segundos epoch
  merchant   : categoria
  is_fraud   : 0 | 1   (muito raro)

Features criadas
  amount_zscore_per_user
  txns_last_1h  (velocidade)
  hour_of_day
  amount_vs_merchant_median

Passos do pipeline
  1. dividir -> treino / valid / teste  (estratificado em is_fraud)
  2. criar features em cada split independentemente
  3. reamostrar só o TREINO (SMOTE | class_weight)
  4. treinar (logreg | árvores com gradient boosting)
  5. avaliar valid: PR-AUC, recall@precisão, matriz de confusão
  6. escolher limiar via curva de custo -> aplicar ao teste
```

## Desafios Extras

- Adicione um detector de anomalia não supervisionado (Isolation Forest) e combine-o com o classificador.
- Simule uma matriz de custo e reporte a perda monetária esperada em cada limiar.
- Adicione validação temporal (treinar no passado, testar no futuro) para pegar concept drift.
- Construa uma fila de alertas simples que ranqueie transações sinalizadas por score.

## Definição de Pronto

- [ ] A divisão acontece antes de qualquer ajuste ou reamostragem — zero vazamento.
- [ ] A reamostragem toca só o fold de treino; validação/teste ficam na prevalência natural.
- [ ] As métricas reportadas incluem PR-AUC e recall a uma precisão declarada, não acurácia.
- [ ] O limiar de decisão é escolhido deliberadamente com um trade-off documentado.
- [ ] Cada transação sinalizada vem com um motivo legível por humanos.

## Armadilhas Comuns

- Aplicar SMOTE antes da divisão, vazando vizinhos sintéticos para a validação.
- Reportar 99% de acurácia numa taxa de fraude de 0,1% e chamar isso de sucesso.
- Deixar o limiar em 0.5 quando o custo de um erro supera de longe um falso alarme (ou vice-versa).
- Criar features usando informação do futuro (ex.: um agregado derivado do rótulo).

## Recursos

- [scikit-learn: métricas de classificação desbalanceada](https://scikit-learn.org/stable/modules/model_evaluation.html#precision-recall-f-measure-metrics) — precisão, recall, curvas PR.
- [Documentação do imbalanced-learn](https://imbalanced-learn.org/stable/) — SMOTE e reamostragem bem feitos.
- [Google: Classificação em dados desbalanceados](https://developers.google.com/machine-learning/data-prep/construct/sampling-splitting/imbalanced-data) — orientação prática.
- [Kaggle: dataset Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) — um conjunto desbalanceado canônico.
