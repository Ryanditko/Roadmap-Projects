# Pipeline de Engenharia de Features

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Modelos são tão bons quanto as features que você fornece, e colunas brutas raramente são as features certas. Este projeto constrói um pipeline reutilizável que transforma dados brutos em features prontas para o modelo: codificar categóricas, escalar numéricas, criar interações e features temporais e então *selecionar* o subconjunto que de fato ajuda. A disciplina que separa isso de uma pilha de transformações ad-hoc é o ajuste sem vazamento — cada transformador aprende seus parâmetros só no fold de treino — e a seleção honesta, em que você prova que um conjunto menor de features se sustenta em dados retidos, em vez de apenas ajustar melhor o ruído do treino.

## Pré-requisitos

- Conforto com uma biblioteca de dataframe e transformadores do scikit-learn (ou equivalente)
- Entender a diferença entre ajustar (fit) e transformar (transform)
- Familiaridade com divisão treino/validação/teste
- Um conjunto tabular com colunas numéricas e categóricas mistas e um alvo

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Construir um pipeline componível de transformadores que ajusta no treino e transforma qualquer split
- Codificar categóricas (one-hot, target/mean com suavização) sem vazar o alvo
- Criar features de interação, polinomiais e derivadas de tempo e julgar quais valem a pena
- Aplicar métodos de seleção filter, wrapper e embedded e compará-los
- Ranquear a importância das features e validar que features podadas não prejudicam o desempenho retido

## Requisitos Funcionais

1. O pipeline deve ser um único objeto ajustado que transforma treino, validação e teste de forma idêntica.
2. Todos os parâmetros dos transformadores (estatísticas do scaler, mapas de codificação) devem ser aprendidos só no treino.
3. Deve gerar pelo menos três tipos de feature (interação, temporal, codificação categórica).
4. Deve implementar pelo menos duas estratégias de seleção e reportar as features que cada uma mantém.
5. Deve produzir uma tabela de importância de features ranqueada.
6. Deve comparar o desempenho do modelo no conjunto completo vs selecionado em dados retidos.
7. Deve tratar valores categóricos não vistos no momento do transform sem quebrar.

## Marcos Sugeridos

1. **Marco 1 — Transformadores:** Construa codificação, escalonamento e geração num só pipeline.
2. **Marco 2 — Seleção:** Adicione e compare estratégias de seleção de features.
3. **Marco 3 — Validar:** Ranqueie importância e confirme que o conjunto selecionado se sustenta no teste.

## Esboço de Dados e Interface

```text
Linha bruta
  id          : string
  age         : int
  signup_date : date
  plan        : categoria
  target      : numérico | classe

Pipeline (ajustar só no TREINO)
  numérico  -> imputar -> escalar
  categoria -> codificar (one-hot | target suavizado)
  temporal  -> days_since, day_of_week, is_weekend
  gerar     -> interações, termos polinomiais
  selecionar-> filter (info mútua) | embedded (L1 / importância de árvore)

Saídas
  matriz de features por split
  tabela de importância: feature -> score
  perf(completo) vs perf(selecionado) em dados retidos
```

## Desafios Extras

- Adicione um feature store leve: persista transformadores ajustados e versione o conjunto de features.
- Detecte e descarte features altamente correlacionadas automaticamente antes de modelar.
- Adicione geração automática de features (ex.: deep feature synthesis) e pode a explosão.
- Rastreie distribuições de features para que uma checagem de drift posterior possa reutilizá-las.

## Definição de Pronto

- [ ] Um pipeline ajustado transforma todos os splits de forma idêntica e reproduzível.
- [ ] Nenhum transformador vê dados de validação ou teste durante o ajuste — sem vazamento do alvo.
- [ ] Pelo menos duas estratégias de seleção são comparadas com as features retidas por cada uma listadas.
- [ ] Uma tabela de importância ranqueada é produzida.
- [ ] O desempenho do conjunto selecionado é comparado ao completo em dados retidos.

## Armadilhas Comuns

- Ajustar o scaler ou o target encoder no conjunto inteiro, vazando estatísticas de teste.
- Fazer target encoding de uma coluna de alta cardinalidade sem suavização ou cálculo out-of-fold.
- Ler a importância das features do ajuste de treino e supor que ela generaliza.
- Quebrar numa categoria não vista na inferência porque o encoder nunca a previu.

## Recursos

- [scikit-learn: Pipelines e estimadores compostos](https://scikit-learn.org/stable/modules/compose.html) — construindo pipelines sem vazamento.
- [scikit-learn: Seleção de features](https://scikit-learn.org/stable/modules/feature_selection.html) — métodos filter, wrapper, embedded.
- [Documentação do category_encoders](https://contrib.scikit-learn.org/category_encoders/) — target encoding feito com segurança.
- [Kaggle: curso de Feature Engineering](https://www.kaggle.com/learn/feature-engineering) — padrões práticos e armadilhas.
