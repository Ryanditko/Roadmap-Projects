# Pipeline de Limpeza de Dados (CSV)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Dados brutos quase nunca estão prontos para análise: colunas têm capitalização inconsistente, números chegam como strings com símbolos de moeda, datas vêm em cinco formatos e algumas linhas estão duplicadas ou pela metade. Neste projeto você constrói um pipeline repetível que pega um CSV bagunçado e produz um limpo e validado — além de um relatório curto descrevendo exatamente o que mudou e por quê. O objetivo não é um notebook cheio de correções manuais, mas um conjunto de passos ordenados e documentados que você pode reexecutar na exportação do mês seguinte e obter o mesmo resultado.

## Pré-requisitos

- Python básico e familiaridade com uma biblioteca de dataframes (pandas ou Polars)
- Entendimento dos tipos de dados comuns (string, inteiro, float, data, booleano)
- Capacidade de ler um CSV e inspecionar suas colunas
- Um conjunto de dados real e bagunçado — o [dataset Titanic do Kaggle](https://www.kaggle.com/c/titanic/data) ou qualquer CSV de dados abertos governamentais funciona bem por ter valores ausentes e tipos mistos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Perfilar um conjunto de dados para encontrar valores ausentes, duplicatas e problemas de tipo
- Escolher e justificar uma estratégia de valores ausentes (descartar, preencher, interpolar) por coluna
- Detectar outliers com uma regra simples e defensável (IQR ou z-score)
- Padronizar rótulos categóricos e normalizar faixas numéricas
- Expressar a limpeza como passos ordenados e reprodutíveis em vez de edições ad-hoc

## Requisitos Funcionais

1. O pipeline deve carregar um CSV de origem e relatar contagem de linhas, de colunas e dtype por coluna.
2. Deve detectar e remover linhas duplicadas exatas, relatando quantas foram removidas.
3. Cada coluna com valores ausentes deve ter uma estratégia de tratamento escolhida explicitamente, não um padrão silencioso.
4. Valores categóricos devem ser padronizados (ex.: `"USA"`, `"usa"`, `" US "` colapsam para um único rótulo).
5. Ao menos uma coluna numérica deve ser verificada quanto a outliers usando uma regra declarada.
6. O pipeline deve gerar um CSV limpo mais um resumo escrito de toda transformação aplicada.
7. Executar o pipeline duas vezes sobre a mesma entrada deve produzir saída idêntica.

## Marcos Sugeridos

1. **Marco 1 — Perfilar:** Carregue o CSV e produza um relatório de qualidade: dimensões, dtypes, contagem de nulos, contagem de duplicatas.
2. **Marco 2 — Limpar:** Aplique remoção de duplicatas, tratamento de valores ausentes e padronização categórica como passos distintos.
3. **Marco 3 — Validar e relatar:** Adicione verificações de outliers e restrições, gere o arquivo limpo e escreva um resumo antes/depois.

## Esboço de Dados e Interface

```text
Relatório de limpeza (por coluna)
  name:          string
  dtype:         tipo inferido
  null_count:    inteiro
  null_strategy: "drop" | "fill_mean" | "fill_mode" | "interpolate" | "keep"
  notes:         string

Estágios do pipeline (ordenados, cada um reexecutável)
  1. load          -> dataframe bruto + perfil
  2. dedupe        -> remove linhas duplicadas exatas
  3. handle_nulls  -> aplica estratégia por coluna
  4. standardize   -> trim, lowercase, mapeia sinônimos categóricos
  5. validate      -> checagens de faixa/restrição, flags de outlier
  6. save          -> cleaned.csv + report.md

Linhas entrada: N   Linhas saída: M   Duplicatas removidas: N-M
```

## Desafios Extras

- Adicione um arquivo de configuração (YAML/JSON) que declare regras por coluna para o pipeline ser orientado a dados, não fixo no código.
- Emita uma pontuação de qualidade de dados antes e depois para quantificar a melhoria.
- Registre linhas rejeitadas ou em quarentena em um arquivo separado em vez de descartá-las silenciosamente.
- Adicione validação de esquema com uma biblioteca como Pandera ou Great Expectations.

## Definição de Pronto

- [ ] O pipeline transforma o CSV bruto em um CSV limpo sem edições manuais no meio do caminho.
- [ ] Toda decisão de valor ausente é explícita e registrada no relatório.
- [ ] Contagens de duplicatas e outliers aparecem no resumo de saída.
- [ ] Rótulos categóricos são consistentes ao longo da coluna limpa.
- [ ] Executar o pipeline duas vezes produz saída verificavelmente igual.

## Armadilhas Comuns

- Preencher valores numéricos ausentes com a média antes de remover outliers, o que distorce a média.
- Descartar linhas com qualquer nulo e perder silenciosamente a maior parte do conjunto — verifique quanto você descarta.
- Padronizar categorias com uma lista manual que quebra no momento em que um novo rótulo aparece.
- Tratar colunas de ID ou CEP como números, fazendo zeros à esquerda sumirem e aplicando matemática a elas.

## Recursos

- [pandas: Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html) — o guia canônico de `dropna`, `fillna` e interpolação.
- [scikit-learn: Imputation of missing values](https://scikit-learn.org/stable/modules/impute.html) — estratégias além de preenchimentos simples.
- [Wikipedia: Interquartile range](https://en.wikipedia.org/wiki/Interquartile_range) — a regra IQR para detecção de outliers.
- [Great Expectations docs](https://docs.greatexpectations.io/) — validação declarativa de dados se você encarar o desafio extra.
