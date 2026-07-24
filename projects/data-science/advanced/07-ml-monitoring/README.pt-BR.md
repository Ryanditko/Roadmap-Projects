# Sistema de Monitoramento de ML

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um sistema de monitoramento que observa modelos em produção e avisa quando eles começam a errar silenciosamente. Um modelo implantado raramente falha de forma barulhenta; ele degrada conforme o mundo se afasta de seus dados de treino, e quando as métricas de negócio caem o estrago já está feito. Este projeto se centra em detectar esse drift cedo: monitorar distribuições de features de entrada, distribuições de predição e — quando os rótulos eventualmente chegam — o desempenho real. Você vai implementar testes estatísticos de drift, ajustar limiares de alerta contra falsos positivos e fechar o ciclo com um gatilho de retreino automático. O entregável é a rede de segurança, não o modelo.

## Pré-requisitos

- Familiaridade com modelos em produção e como eles degradam
- Entendimento de distâncias e testes estatísticos (PSI, teste KS, qui-quadrado)
- Experiência com uma stack de métricas/dashboard (Prometheus + Grafana ou similar)
- Conforto com dados de série temporal e rótulos de verdade atrasados

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Distinguir drift de dados, drift de conceito e atraso de rótulo, e monitorar cada um apropriadamente
- Implementar detecção estatística de drift em features e predições
- Calcular o desempenho real assim que os rótulos atrasados chegam
- Ajustar limiares de alerta para equilibrar detecção precoce contra falsos positivos
- Disparar retreino automático a partir de um sinal de monitoramento

## Requisitos Funcionais

1. O sistema deve registrar entradas do modelo, predições e (quando disponíveis) rótulos de verdade.
2. Deve calcular drift de feature por feature contra uma distribuição de referência/treino.
3. Deve monitorar a distribuição de predição em busca de mudanças independentes dos rótulos.
4. Quando os rótulos chegam, deve calcular métricas de desempenho sobre a janela correspondente.
5. Deve emitir alertas quando o drift ou o desempenho cruzam limiares configuráveis.
6. Deve apresentar dashboards mostrando tendências de drift e desempenho ao longo do tempo.
7. Deve expor um gatilho de retreino que dispara em uma condição de alerta sustentada.

## Requisitos Não Funcionais

- **Tempestividade:** o drift em uma feature monitorada deve ser detectável dentro de uma janela limitada.
- **Controle de falsos positivos:** os limiares de alerta devem ser ajustáveis e a taxa de falsos positivos reportada.
- **Escalabilidade:** o monitoramento deve lidar com muitas features e múltiplos modelos sem explosão linear de custo.
- **Reprodutibilidade:** a distribuição de referência e os limiares devem ser versionados.

## Marcos Sugeridos

1. **Marco 1 — Registro:** Capture entradas, predições e rótulos com timestamps e versão do modelo.
2. **Marco 2 — Detecção de drift:** Implemente testes de drift de feature e predição contra uma referência.
3. **Marco 3 — Desempenho e alertas:** Calcule o desempenho com rótulos atrasados e adicione alertas por limiar.
4. **Marco 4 — Dashboards e retreino:** Construa dashboards de tendência e conecte um gatilho de retreino.

## Esboço de Dados e Interface

```text
 serving  --> log de predição { ts, model_version, features{}, pred, [label] }
                     |
                     v
 +--------------------------+   janela vs distribuição de referência
 | Detectores de drift      |   feature: PSI / KS ;  pred: mudança populacional
 +------------+-------------+
              v
 +--------------------------+   chega atrasado; unido por request_id
 | Cálculo perf (rótulos)   |   acurácia, AUC, calibração na janela
 +------------+-------------+
              v
 +--------------------------+   limiares (versionados)
 | Alertas                  |   violação sustentada -> notifica + dispara
 +------------+-------------+
              v
     Dashboards (tendências drift/perf)   +   POST /retrain (em alerta sustentado)

 Métricas por feature: PSI, distância-à-referência, taxa-de-ausência
 Referência: distribuição do período de treino congelada, versionada
```

## Desafios Extras

- Adicione segmentação automática para achar qual fatia está com drift, não apenas que há drift.
- Suporte detecção de drift multivariada, não apenas por feature.
- Adicione um detector de "falha silenciosa" usando a confiança da predição quando faltam rótulos.
- Rastreie drift entre múltiplas versões de modelo em um único dashboard.

## Definição de Pronto

- [ ] Entradas, predições e rótulos são registrados e uníveis por request ID.
- [ ] Drift de feature e de predição são calculados contra uma distribuição de referência versionada.
- [ ] O desempenho com rótulos atrasados é calculado sobre a janela de tempo correta.
- [ ] Alertas disparam em violações de limiar sustentadas, com uma taxa de falsos positivos reportada.
- [ ] Um alerta sustentado pode disparar um job de retreino automaticamente.

## Armadilhas Comuns

- Monitorar apenas a acurácia e ficar cego até os rótulos chegarem semanas depois.
- Alertar em cada pequena oscilação de distribuição, treinando a equipe a ignorar alertas.
- Comparar contra uma referência obsoleta e tratar sazonalidade normal como drift.
- Não unir predições a rótulos corretamente, tornando os números de desempenho errados.

## Recursos

- [Google: Data Validation para Machine Learning / TFX](https://www.tensorflow.org/tfx/guide/tfdv) — detecção de skew de distribuição e schema.
- [Documentação do Evidently AI](https://docs.evidentlyai.com/) — métricas de drift e relatórios de monitoramento.
- [A Survey on Concept Drift Adaptation (Gama et al., 2014)](https://dl.acm.org/doi/10.1145/2523813) — tipos de drift e métodos de detecção.
- [Population Stability Index (PSI) explicado](https://www.mdpi.com/2227-9091/7/2/53) — uma métrica de drift padrão para features tabulares.
