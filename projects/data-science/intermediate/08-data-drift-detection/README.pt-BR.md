# Detecção de Data Drift

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Um modelo treinado nos dados do ano passado apodrece silenciosamente quando os dados deste ano deixam de se parecer com eles. Este projeto constrói um monitor que observa os dados que chegam contra uma distribuição de referência (de treino) e levanta um alerta quando elas divergem — antes que as predições do modelo degradem sem aviso. Você vai implementar testes de distância estatística por feature, distinguir drift de feature de drift de predição, definir limiares que separam mudança genuína de ruído comum e produzir um relatório interpretável que diz não só *que* houve drift, mas *quais* features se moveram e por quanto. O eixo metodológico é uma janela de referência limpa e congelada, comparada contra janelas deslizantes de produção.

## Pré-requisitos

- Conforto com uma biblioteca de dataframe e estatística básica (distribuições, quantis)
- Entender a distinção entre dados de treino e de serving
- Familiaridade com teste de hipóteses (p-valores, estatísticas de teste)
- Um conjunto que você possa dividir em um período de referência e lotes posteriores de "produção"

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Congelar uma distribuição de referência a partir dos dados de treino e comparar lotes contra ela
- Detectar drift numérico com o teste de Kolmogorov-Smirnov e drift categórico com qui-quadrado ou PSI
- Calcular o Population Stability Index e interpretar seus limiares convencionais
- Separar drift de feature (covariável) de drift de predição/alvo e drift de rótulo
- Definir limiares cientes do ruído para que a variação comum não dispare falsos alarmes constantes

## Requisitos Funcionais

1. O monitor deve aceitar um conjunto de referência e lotes sucessivos de produção.
2. Deve calcular uma estatística de drift por feature apropriada ao tipo da feature.
3. Deve reportar o Population Stability Index por feature com uma faixa de severidade (estável/moderado/severo).
4. Deve distinguir drift de feature de entrada de drift da distribuição de predição.
5. Deve aplicar limiares configuráveis e emitir um alerta só quando eles são ultrapassados.
6. Deve ranquear quais features sofreram mais drift em um lote dado.
7. Deve rastrear drift ao longo do tempo para que tendências sejam visíveis, não só snapshots pontuais.

## Marcos Sugeridos

1. **Marco 1 — Referência e testes:** Congele a referência e implemente testes de drift por feature.
2. **Marco 2 — PSI e alertas:** Adicione PSI, faixas de severidade e alertas por limiar.
3. **Marco 3 — Rastrear e ranquear:** Registre drift ao longo dos lotes e ranqueie os que mais moveram.

## Esboço de Dados e Interface

```text
Janela de referência (dos dados de treino)
  por feature numérica  : histograma / quantis
  por feature categórica: frequências de categoria

Checagem de lote
  para cada feature:
    numérica    -> estatística do teste KS, p_value
    categórica  -> qui-quadrado | PSI
  psi = soma( (p_i - q_i) * ln(p_i / q_i) )
  faixa: psi<0.1 estável | 0.1-0.25 moderado | >0.25 severo

Relatório
  batch_id, ts
  feature -> {psi, test_p, faixa}
  prediction_drift: {psi na saída do modelo}
  alerta: any(faixa == severo)
  top_movers: features ordenadas por psi desc
```

## Desafios Extras

- Adicione detecção de drift multivariado (ex.: um classificador de domínio que tenta separar referência de lote).
- Correlacione o drift detectado com uma queda observada de desempenho onde os rótulos chegam atrasados.
- Adicione atualização automática da janela de referência com política documentada e guardrails.
- Construa um pequeno dashboard de séries temporais de PSI por feature.

## Definição de Pronto

- [ ] Uma referência congelada é comparada contra cada lote de produção, nunca reajustada silenciosamente.
- [ ] Features numéricas e categóricas usam, cada uma, um teste de drift apropriado.
- [ ] O PSI é reportado por feature com faixas de severidade.
- [ ] Drift de feature e drift de predição são reportados separadamente.
- [ ] Alertas disparam só além dos limiares configurados e os top movers são ranqueados.

## Armadilhas Comuns

- Comparar contra uma referência que fica se atualizando, mascarando o drift real.
- Sinalizar todo p-valor minúsculo em lotes enormes, onde até mudanças triviais são "significativas".
- Observar só as features de entrada e não perceber que a mistura de predições mudou.
- Usar um limiar global único para features com variâncias naturais muito diferentes.

## Recursos

- [Evidently AI: guia de data drift](https://docs.evidentlyai.com/) — detecção prática de drift e relatórios.
- [Wikipedia: teste de Kolmogorov–Smirnov](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) — o cavalo de batalha do drift numérico.
- [Population Stability Index explicado](https://www.listendata.com/2015/05/population-stability-index.html) — fórmula e limiares do PSI.
- [Google: drift de dados e conceito em ML](https://developers.google.com/machine-learning/managing-ml-projects/monitoring) — monitoramento em produção.
