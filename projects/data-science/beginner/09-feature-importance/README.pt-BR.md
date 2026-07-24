# Análise de Importância de Features

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Um modelo que prevê bem mas não consegue se explicar é difícil de vender para quem precisa agir sobre ele. Importância de features é como você responde "quais entradas de fato dirigem a previsão?" — e é mais traiçoeiro do que parece, porque métodos diferentes podem discordar e features correlacionadas podem roubar o crédito umas das outras. Neste projeto você treina um modelo em um conjunto tabular, ranqueia suas features com ao menos dois métodos de importância, compara os rankings e transforma o resultado em recomendações em linguagem simples. O objetivo é interpretação que você consiga defender, não um único ranking aceito por fé.

## Pré-requisitos

- Python básico, pandas e scikit-learn
- Ter treinado ao menos um modelo antes (veja [Modelo de Regressão Linear](../03-linear-regression/) ou [Modelo de Classificação](../04-classification-iris/))
- Entendimento de features e um alvo
- Um conjunto tabular com várias features — o [dataset Wine Quality da UCI](https://archive.ics.uci.edu/dataset/186/wine+quality) ou o [dataset diabetes do scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_diabetes.html) funcionam bem

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Calcular importância de features com mais de um método
- Explicar como importâncias baseadas em modelo, por permutação e SHAP diferem
- Reconhecer como features correlacionadas distorcem os rankings de importância
- Visualizar e comunicar importância com clareza
- Traduzir um ranking em recomendações acionáveis e honestamente ressalvadas

## Requisitos Funcionais

1. O fluxo deve treinar um modelo preditivo em um conjunto tabular.
2. Deve calcular importância de features com ao menos dois métodos distintos.
3. Deve apresentar cada ranking como um gráfico de barras ordenado e rotulado.
4. Deve comparar os rankings e discutir onde e por que discordam.
5. Deve identificar ao menos um par de features correlacionadas e notar o efeito na importância.
6. Deve produzir um ranking de importância por permutação calculado em dados separados.
7. Deve terminar com recomendações em linguagem simples sobre quais features importam.

## Marcos Sugeridos

1. **Marco 1 — Treinar:** Ajuste um modelo bom o bastante para interpretar e confirme seu desempenho de baseline.
2. **Marco 2 — Ranquear:** Calcule importância baseada em modelo e por permutação; visualize ambas.
3. **Marco 3 — Reconciliar:** Compare rankings, inspecione correlações e escreva as recomendações.

## Esboço de Dados e Interface

```text
Métodos de importância (todos retornam: feature -> escore)
  baseada em modelo  feature_importances_ de árvore ou |coef| linear  (rápida, pode enviesar)
  permutação         embaralha uma coluna, mede a queda de desempenho   (agnóstica ao modelo)
  SHAP               contribuição por previsão, média                   (detalhada, mais lenta)

Tabela de ranking (comparar lado a lado)
  feature        model_imp   perm_imp   rank_concorda?
  alcohol          0.28        0.31        sim
  density          0.19        0.05        NAO  <- provavelmente correlacionada c/ alcohol
  citric_acid      0.04        0.03        sim

Checagem de correlação
  corr(density, alcohol) = -0.69  -> importância pode ser dividida/roubada entre elas
```

## Desafios Extras

- Adicione valores SHAP e compare seu ranking com a importância por permutação.
- Adicione gráficos de dependência parcial para as duas features do topo para mostrar a direção do efeito.
- Remova as features pior ranqueadas e cheque se o desempenho sobrevive.
- Repita a análise com um segundo tipo de modelo e veja se as features do topo são estáveis.

## Definição de Pronto

- [ ] Existe um modelo treinado com uma pontuação de baseline declarada para interpretar.
- [ ] Ao menos dois métodos de importância são calculados e plotados.
- [ ] A importância por permutação é calculada em dados separados, não de treino.
- [ ] Discordâncias entre rankings são explicadas, incluindo um efeito de correlação.
- [ ] As recomendações são escritas em linguagem simples com as ressalvas apropriadas.

## Armadilhas Comuns

- Confiar só no `feature_importances_` baseado em árvore — ele é enviesado para features de alta cardinalidade.
- Calcular importância por permutação nos dados de treino, o que recompensa memorização.
- Ler importância como causalidade ("esta feature causa o resultado").
- Dividir a importância entre duas features correlacionadas e concluir que ambas são fracas.

## Recursos

- [scikit-learn: Permutation feature importance](https://scikit-learn.org/stable/modules/permutation_importance.html) — o método agnóstico ao modelo e suas ressalvas.
- [scikit-learn: Feature importances caveats](https://scikit-learn.org/stable/auto_examples/inspection/plot_permutation_importance.html) — por que a importância por impureza engana.
- [SHAP documentation](https://shap.readthedocs.io/) — explicações aditivas por previsão.
- [Interpretable Machine Learning (Molnar)](https://christophm.github.io/interpretable-ml-book/) — um livro gratuito sobre importância e interpretação.
