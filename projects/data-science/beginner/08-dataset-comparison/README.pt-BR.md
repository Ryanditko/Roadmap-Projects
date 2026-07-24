# Ferramenta de Comparação de Datasets

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Quando um pipeline de dados quebra silenciosamente, o sintoma costuma ser um conjunto de dados que mudou de forma sem avisar: uma coluna sumiu, uma distribuição deslocou, nulos apareceram. Neste projeto você constrói uma ferramenta que recebe dois conjuntos de dados — pense "mês passado vs este mês" ou "dados de treino vs dados de produção" — e relata exatamente como diferem. Ela perfila cada um, compara esquemas e estatísticas lado a lado, sinaliza deriva (drift) de distribuição e produz um relatório legível. Essa é a base do monitoramento e validação de dados, e ensina você a descrever um conjunto com precisão suficiente para notar quando ele muda.

## Pré-requisitos

- Python básico e pandas
- Estatística descritiva básica (média, mediana, desvio padrão, quantis)
- Entendimento do esquema de um conjunto de dados (nomes e tipos de colunas)
- Dois conjuntos de dados relacionados para comparar — dois snapshots do mesmo dataset do [Kaggle](https://www.kaggle.com/datasets) ao longo do tempo, ou um conjunto que você divide e perturba de propósito

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Perfilar um conjunto de dados em um resumo compacto e comparável
- Comparar dois esquemas e detectar colunas adicionadas, removidas ou com tipo alterado
- Comparar distribuições de colunas comuns e quantificar a deriva
- Escolher um teste estatístico simples para checar se duas amostras diferem
- Gerar um relatório de comparação claro, lado a lado

## Requisitos Funcionais

1. A ferramenta deve carregar dois conjuntos de dados e perfilar cada um (dimensões, colunas, dtypes, taxa de nulos).
2. Deve relatar diferenças de esquema: colunas só em A, só em B e incompatibilidades de tipo.
3. Para colunas numéricas compartilhadas, deve comparar estatísticas de resumo lado a lado.
4. Deve sinalizar colunas cuja distribuição deslocou além de um limiar escolhido.
5. Deve aplicar ao menos um teste estatístico (ex.: Kolmogorov–Smirnov ou qui-quadrado) a uma coluna compartilhada.
6. Deve tratar conjuntos com colunas parcialmente sobrepostas sem travar.
7. Deve gerar um único relatório de comparação legível.

## Marcos Sugeridos

1. **Marco 1 — Perfilar:** Construa um resumo de profiling para cada conjunto independentemente.
2. **Marco 2 — Comparar:** Faça o diff dos esquemas e coloque as estatísticas das colunas comuns lado a lado.
3. **Marco 3 — Detectar deriva e relatar:** Adicione um teste de distribuição, sinalize a deriva e monte o relatório.

## Esboço de Dados e Interface

```text
Perfil por dataset
  linhas, colunas
  por coluna: { name, dtype, null_rate, n_unique, mean?, std?, min?, max? }

Saída da comparação
  diff de esquema
    only_in_A:   [ ... ]
    only_in_B:   [ ... ]
    type_change: [ { col, type_A, type_B } ]
  colunas numéricas compartilhadas (lado a lado)
    col        mean_A   mean_B   std_A   std_B   null_A   null_B   drift?
    age        38.1     41.7     11.2    12.9    0.0%     3.4%     YES
  teste de distribuição
    KS(col) -> estatística, p_value -> "distribuições diferem" se p < 0.05
```

## Desafios Extras

- Adicione uma "pontuação de diferença" numérica por coluna e ranqueie as colunas pelo quanto mudaram.
- Visualize distribuições sobrepostas para as colunas com maior deriva.
- Compare colunas categóricas pela mudança de frequência de valores, não só pela presença.
- Empacote a ferramenta para rodar em agenda e alertar quando a deriva exceder um limiar.

## Definição de Pronto

- [ ] Ambos os conjuntos são perfilados com campos de resumo correspondentes.
- [ ] Diferenças de esquema (colunas adicionadas, removidas, com tipo alterado) são relatadas.
- [ ] Colunas numéricas compartilhadas são comparadas estatística por estatística.
- [ ] Ao menos um teste de distribuição é aplicado e seu resultado interpretado.
- [ ] A ferramenta roda em conjuntos com colunas apenas parcialmente sobrepostas.

## Armadilhas Comuns

- Assumir que ambos os conjuntos compartilham toda coluna e travar na primeira incompatibilidade.
- Comparar só médias, perdendo uma mudança de variância ou de forma que deixa a média intacta.
- Ler o p-valor de um teste estatístico como "o quão grande" é a diferença (não é).
- Tratar diferenças minúsculas de tamanho de amostra como deriva quando são só ruído.

## Recursos

- [pandas: DataFrame.describe](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.describe.html) — o perfil estatístico rápido.
- [SciPy: Kolmogorov–Smirnov test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ks_2samp.html) — comparando duas distribuições contínuas.
- [SciPy: Chi-square test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.chi2_contingency.html) — comparando distribuições categóricas.
- [Evidently AI: Data drift](https://docs.evidentlyai.com/) — como o monitoramento de deriva em produção formaliza essa ideia.
