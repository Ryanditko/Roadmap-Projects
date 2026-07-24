# Ferramenta de IA Explicável

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa uma ferramenta que explica por que um modelo fez uma dada predição, tanto no nível de uma única predição quanto no nível do modelo inteiro. Conforme os modelos entram em decisões que afetam pessoas — crédito, contratação, saúde — "o modelo disse" não é uma resposta aceitável, e em algumas jurisdições também não é uma resposta legal. Neste projeto você implementa os métodos centrais de explicabilidade (SHAP, LIME, contrafactuais), raciocina sobre quando cada um é confiável, sobrepõe uma análise de justiça/viés e apresenta tudo de forma que um stakeholder não técnico consiga agir. A parte difícil não é calcular uma explicação — é fazer uma que seja fiel e honesta.

## Pré-requisitos

- Familiaridade com famílias de modelos comuns (árvores, gradient boosting, redes neurais)
- Entendimento de atribuição de features e métodos agnósticos vs específicos de modelo
- Experiência com uma biblioteca de plots/dashboard
- Conforto para discutir métricas de justiça e seus tradeoffs

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Produzir explicações locais com SHAP e LIME e conhecer suas suposições e limites
- Produzir explicações globais (importância de feature, dependência) fiéis ao modelo
- Gerar explicações contrafactuais acionáveis ("mude X para inverter o resultado")
- Rodar uma análise de viés/justiça entre grupos protegidos
- Comunicar explicações a um público não técnico sem exagerar nas afirmações

## Requisitos Funcionais

1. A ferramenta deve aceitar um modelo treinado e dataset e produzir explicações locais para predições individuais.
2. Deve produzir explicações globais resumindo a influência geral das features.
3. Deve implementar ao menos dois métodos (ex.: SHAP e LIME) e permitir ao usuário compará-los.
4. Deve gerar explicações contrafactuais descrevendo mudanças mínimas de entrada para alterar uma predição.
5. Deve calcular métricas de justiça entre ao menos um atributo protegido.
6. Deve apresentar explicações visualmente de forma que um não-especialista consiga interpretar.
7. Deve sinalizar quando uma explicação é não confiável (ex.: extrapolação fora do manifold dos dados).

## Requisitos Não Funcionais

- **Fidelidade:** as explicações devem ser validadas contra um método de verdade quando existir (ex.: SHAP exato para árvores).
- **Latência:** explicações locais devem ser produzidas dentro de um orçamento interativo para o modelo alvo.
- **Reprodutibilidade:** explicações para a mesma entrada e modelo devem ser estáveis entre execuções (com semente).
- **Auditabilidade:** cada explicação deve ser registrada com versão do modelo e entrada.

## Marcos Sugeridos

1. **Marco 1 — Explicações locais:** Implemente SHAP e LIME para predições únicas com saída visual.
2. **Marco 2 — Visão global:** Adicione importância global de features e plots de dependência.
3. **Marco 3 — Contrafactuais e justiça:** Gere contrafactuais e calcule métricas de justiça por grupo.
4. **Marco 4 — Confiança e apresentação:** Adicione flags de confiabilidade e um relatório/dashboard voltado ao stakeholder.

## Esboço de Dados e Interface

```text
 modelo treinado + dataset
        |
        v
 +-------------------------+   escolha uma instância x
 | Explicadores locais     |
 |   SHAP(x)  ->  phi_i     |   contribuição por feature
 |   LIME(x)  ->  pesos     |   surrogate local
 |   comparar + concordância|
 +-----------+-------------+
             |
 +-------------------------+   sobre o dataset
 | Explicadores globais     |   média|phi|, plots de dependência
 +-----------+-------------+
             |
 +-------------------------+   delta mínimo para inverter a predição
 | Busca contrafactual      |   "aumente a renda em X -> aprovado"
 +-----------+-------------+
             |
 +-------------------------+   por grupo protegido
 | Análise de justiça       |   paridade demográfica, gap de opp. igual
 +-----------+-------------+
             v
  Relatório/dashboard  +  flag de confiabilidade (na distribuição?)  +  log de auditoria
```

## Desafios Extras

- Adicione explicações por âncora (regras if-then de alta precisão) junto de SHAP/LIME.
- Adicione explicações baseadas em exemplos (protótipos e críticas, pontos de treino influentes).
- Suporte explicação para modelos de imagem ou texto, não apenas tabulares.
- Adicione um fluxo de "contestar esta decisão" que exibe o contrafactual ao usuário afetado.

## Definição de Pronto

- [ ] Explicações locais (SHAP e LIME) são produzidas e comparadas visualmente.
- [ ] Importância global de features e dependência são mostradas fielmente ao modelo.
- [ ] Contrafactuais descrevem mudanças de entrada mínimas e viáveis para inverter uma predição.
- [ ] Métricas de justiça são calculadas para ao menos um grupo protegido.
- [ ] As explicações são sinalizadas quando a entrada está fora da distribuição de treino.

## Armadilhas Comuns

- Apresentar pesos de LIME/SHAP como causais quando são apenas associativos.
- Explicar uma única predição e generalizá-la para o comportamento do modelo inteiro.
- Produzir contrafactuais matematicamente válidos mas praticamente impossíveis (ex.: "reduza sua idade").
- Reportar uma métrica de justiça como se justiça fosse um único número, escondendo os tradeoffs.

## Recursos

- [Documentação do SHAP](https://shap.readthedocs.io/en/latest/) — atribuição unificada de features baseada em valores de Shapley.
- [A Unified Approach to Interpreting Model Predictions (Lundberg & Lee, 2017)](https://arxiv.org/abs/1705.07874) — o paper do SHAP.
- ["Why Should I Trust You?": Explaining the Predictions of Any Classifier (Ribeiro et al., 2016)](https://arxiv.org/abs/1602.04938) — o paper do LIME.
- [Interpretable Machine Learning (Molnar) — livro online](https://christophm.github.io/interpretable-ml-book/) — métodos, suposições e armadilhas.
