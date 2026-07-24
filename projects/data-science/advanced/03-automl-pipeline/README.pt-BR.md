# Pipeline de AutoML

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um sistema que, dado um dataset tabular e uma coluna alvo, busca sobre escolhas de pré-processamento, algoritmos candidatos e hiperparâmetros para retornar um bom modelo quase sem ajuste manual. AutoML é enganosamente profundo: o espaço de busca é enorme, a computação é finita e é perigosamente fácil fazer overfitting no conjunto de validação por seleção repetida. O trabalho interessante é a estratégia de busca (otimização Bayesiana vence grid/random por um motivo), a avaliação honesta sob um orçamento de computação e um relatório em que um humano possa realmente confiar. Você está construindo o motor, não chamando um pronto de prateleira.

## Pré-requisitos

- Domínio sólido de validação cruzada, vazamento de dados e o tradeoff viés–variância
- Experiência com pipelines do scikit-learn e várias famílias de modelos
- Familiaridade conceitual com uma biblioteca de otimização (Optuna ou Hyperopt)
- Conforto para raciocinar sobre orçamentos de computação e jobs paralelos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Definir um espaço de busca estruturado sobre pré-processamento, algoritmos e hiperparâmetros
- Implementar e comparar estratégias de busca (random, Bayesiana, successive halving)
- Usar early stopping e pruning para gastar computação onde ela compensa
- Proteger-se contra overfitting no conjunto de validação com CV aninhada ou um portão held-out
- Produzir um leaderboard e uma análise que um stakeholder consiga interpretar

## Requisitos Funcionais

1. O sistema deve aceitar um dataset e alvo e inferir tipos de colunas (numérica, categórica, datetime).
2. Deve construir pipelines de pré-processamento por tipo de coluna como parte da busca.
3. Deve buscar sobre ao menos três famílias de algoritmos com seus hiperparâmetros.
4. A busca deve suportar uma estratégia Bayesiana e respeitar um orçamento de tempo de relógio ou de trials.
5. Trials com baixo desempenho devem ser podados/interrompidos cedo em vez de rodar até o fim.
6. A seleção final do modelo deve usar um split de avaliação não tocado durante a busca.
7. O sistema deve produzir um leaderboard ranqueado com métricas e a configuração do pipeline vencedor.

## Requisitos Não Funcionais

- **Reprodutibilidade:** uma semente e um orçamento fixos devem reproduzir o mesmo leaderboard.
- **Vazão:** os trials devem rodar em paralelo e escalar com os workers disponíveis.
- **Robustez:** um único trial que falha não deve abortar toda a busca.
- **Aderência ao orçamento:** a busca deve parar dentro do limite de tempo/trials configurado.

## Marcos Sugeridos

1. **Marco 1 — Inferência de tipos e pré-processamento:** Infira tipos de colunas e construa pipelines de pré-processamento por tipo.
2. **Marco 2 — Motor de busca:** Conecte um otimizador sobre uma família de algoritmos com CV adequada.
3. **Marco 3 — Multi-algoritmo e pruning:** Adicione mais famílias, busca Bayesiana e early stopping.
4. **Marco 4 — Portão honesto e leaderboard:** Adicione um portão de seleção held-out e um leaderboard reportável.

## Esboço de Dados e Interface

```text
 dataset + alvo
        |
        v
 +----------------+    infere  {col -> numeric|categorical|datetime}
 | Inferência tipo|
 +-------+--------+
         v
 +----------------------------+
 | Controlador busca (Optuna) |  orçamento: N trials ou T minutos
 |   amostra -> monta pipeline|
 |   -> score CV -> podar?    |
 +-------------+--------------+
    workers paralelos | trials
                       v
 +----------------------------+
 | Trial: preproc + modelo    |  famílias: {árvores, linear, boosting}
 +-------------+--------------+
               v
 +----------------------------+
 | Portão de seleção held-out |  <- split não tocado
 +-------------+--------------+
               v
   Leaderboard[ {rank, algo, params, cv_score, holdout_score} ]

Anti-vazamento: ajuste o pré-processamento DENTRO de cada fold da CV, nunca nos dados completos
```

## Desafios Extras

- Adicione warm starts por meta-aprendizado de execuções anteriores em datasets similares.
- Suporte busca multi-objetivo (acurácia vs latência de inferência) com uma fronteira de Pareto.
- Adicione geração automática de features (interações, target encoding) como dimensões de busca.
- Persista e retome uma busca interrompida a partir do histórico de trials.

## Definição de Pronto

- [ ] O pré-processamento é ajustado dentro de cada fold da CV, sem vazamento do dataset completo.
- [ ] Ao menos três famílias de algoritmos são buscadas com uma estratégia Bayesiana.
- [ ] Pruning/early stopping comprovadamente economiza computação versus busca exaustiva.
- [ ] A seleção final usa um split não tocado durante a busca, e seu score é reportado separadamente.
- [ ] Uma semente e um orçamento fixos reproduzem o mesmo leaderboard.

## Armadilhas Comuns

- Ajustar scalers ou encoders no dataset completo antes da CV, vazando informação de teste.
- Selecionar o "melhor" modelo nos mesmos folds de validação usados para tuning e depois superestimar sua acurácia.
- Deixar um trial que quebra matar toda a execução em vez de registrar e pular.
- Ignorar o orçamento de computação e reportar resultados que levaram dez vezes mais tempo que o permitido.

## Recursos

- [Documentação do Optuna](https://optuna.readthedocs.io/en/stable/) — busca Bayesiana, pruners e persistência de estudos.
- [scikit-learn: armadilhas comuns e práticas recomendadas](https://scikit-learn.org/stable/common_pitfalls.html) — vazamento e avaliação feitos direito.
- [Auto-sklearn (Feurer et al., 2015)](https://papers.nips.cc/paper/2015/hash/11d0e6287202fced83f79975ec59a3a6-Abstract.html) — meta-aprendizado e construção de ensembles em AutoML.
- [Hyperband (Li et al., 2018)](https://jmlr.org/papers/v18/16-558.html) — successive halving para busca consciente de orçamento.
