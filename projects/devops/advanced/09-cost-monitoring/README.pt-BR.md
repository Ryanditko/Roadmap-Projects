# Sistema de Monitoramento de Custos

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Transforme uma fatura de nuvem opaca em uma visão acionável, atribuível e prospectiva do gasto. Você vai ingerir dados de custo e uso, alocá-los a times e serviços via tagueamento, expor desperdício e anomalias, prever para onde a fatura está indo, e colocar salvaguardas para que um recurso descontrolado seja pego antes de aparecer como uma surpresa de cinco dígitos. A substância avançada está na alocação e nos incentivos: dados de custo são bagunçados, tags são inconsistentes, recursos compartilhados resistem a um chargeback limpo, e "só desliga" ignora a confiabilidade que o gasto compra. Um bom sistema de custos não apenas reporta números — ele os atribui a donos, explica a tendência, e torna o custo de uma decisão visível no momento em que a decisão é tomada.

## Pré-requisitos

- Acesso a uma exportação de custo e uso de um provedor de nuvem (ou um dataset de amostra realista)
- Entendimento do seu inventário de recursos: computação, armazenamento, rede, serviços gerenciados
- Familiaridade com tagueamento/labeling e como ele guia a alocação
- Habilidades básicas de modelagem de dados e dashboards

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Ingerir e normalizar dados de custo e uso de nuvem
- Alocar gasto a times/serviços via uma estratégia de tags, incluindo custos compartilhados
- Detectar desperdício (recursos ociosos, superdimensionados, órfãos) e anomalias de custo
- Prever gasto e comparar com orçamentos
- Definir salvaguardas e alertas que pegam custo descontrolado cedo sem bloquear trabalho legítimo

## Requisitos Funcionais

1. O sistema deve ingerir dados de custo e uso de forma agendada e armazená-los de modo consultável.
2. O gasto deve ser alocável a um time ou serviço, com uma regra definida para custos compartilhados/sem tag.
3. O sistema deve sinalizar desperdício: recursos ociosos, superdimensionados ou órfãos.
4. Anomalias de custo (picos súbitos vs. baseline) devem ser detectadas e alertadas.
5. O sistema deve prever o gasto do período atual e comparar com um orçamento.
6. Uma violação de orçamento ou previsão-de-violação deve disparar um alerta ao dono.
7. Relatórios devem ser atribuíveis — todo custo significativo tem um dono ou um balde compartilhado documentado.

## Marcos Sugeridos

1. **Marco 1 — Ingerir e modelar:** Carregue dados de custo/uso e modele-os para consulta.
2. **Marco 2 — Alocação:** Aplique uma estratégia de tags e atribua gasto a donos, tratando custos compartilhados.
3. **Marco 3 — Desperdício e anomalias:** Detecte recursos ociosos/superdimensionados e anomalias de pico.
4. **Marco 4 — Previsão e salvaguardas:** Preveja o período, compare com orçamentos e alerte sobre risco de violação.

## Esboço de Dados e Interface

```text
   exportação de billing da nuvem (diária)
      │
      ▼
   ┌───────────────┐   normaliza + alocação baseada em tags
   │ Ingestor de    │──────────────┐
   │ custo          │              ▼
   └───────────────┘       ┌───────────────┐
                           │ Store de custo │  (serviço, time, recurso, $)
                           └──────┬────────┘
        ┌──────────────┬──────────┼───────────┬──────────────┐
        ▼              ▼          ▼           ▼              ▼
   alocação        localizador  detector   previsão       salvaguarda
   por time/svc    de desperdício de anomalia (período $)  de orçamento
        └──────────────┴──────────┴───────────┴──────────────┘
                            ▼
                     ┌───────────┐   alerta dono (name@example.com)
                     │ dashboard  │   em anomalia / violação de orçamento
                     └───────────┘

Registro de alocação:
  serviço: checkout   time: payments   custo: $/dia
  compartilhado: load-balancer -> dividido por fatia de requisições (regra documentada)

Metas não-funcionais:
  cobertura de atribuição  >= 95% do gasto mapeado a um dono
  detecção de anomalia     pico sinalizado em até 24h
  acurácia de previsão     monitorada (delta previsão vs. real)
```

## Desafios Extras

- Adicione um relatório de chargeback/showback por time com tendências e principais causas.
- Recomende oportunidades de rightsizing e compromisso (reserved/savings-plan) com estimativas de payback.
- Adicione métricas de economia unitária (custo por requisição, por tenant) para decisões de escala cientes de custo.
- Suporte multi-cloud para que alocação e anomalias atravessem provedores.

## Definição de Pronto

- [ ] Dados de custo/uso são ingeridos de forma agendada e consultáveis.
- [ ] Pelo menos 95% do gasto é atribuído a um dono, com uma regra documentada para custo compartilhado.
- [ ] Desperdício e anomalias de custo são detectados e alertados.
- [ ] O gasto é previsto para o período e comparado com um orçamento.
- [ ] Um risco de violação de orçamento dispara um alerta ao dono.

## Armadilhas Comuns

- Perseguir totais brutos de custo sem alocação, de modo que ninguém é dono nem age sobre o número.
- Tagueamento inconsistente que deixa um grande balde "não alocado" que ninguém investiga.
- Alertas de anomalia sem baseline, disparando em padrões normais de fim de mês ou de escala.
- Prever com base em uma janela curta, então a sazonalidade torna toda previsão errada.
- Otimizar custo isoladamente e degradar silenciosamente a confiabilidade que o gasto comprava.

## Recursos

- [FinOps Framework](https://www.finops.org/framework/) — a disciplina de gestão financeira de nuvem.
- [AWS Well-Architected: Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) — padrões de alocação e rightsizing.
- [Google Cloud: Cost management overview](https://cloud.google.com/cost-management) — conceitos de orçamentos, alertas e relatórios.
- [Kubecost / OpenCost](https://www.opencost.io/) — alocação de custo open-source para Kubernetes.
