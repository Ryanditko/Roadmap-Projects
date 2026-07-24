# Automação de Resposta a Incidentes

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa a maquinaria que transforma um alerta bruto em uma resposta coordenada: a detecção roteia para o on-call certo, um canal de incidente e uma linha do tempo são criados automaticamente, remediações seguras de primeira linha rodam sem esperar por um humano, e tudo é registrado para um post-mortem sem culpa. O objetivo é encolher os dois números que definem a dor de um incidente — tempo para detectar e tempo para resolver — mantendo um humano firmemente no controle de qualquer coisa arriscada. As partes difíceis e interessantes são os julgamentos codificados em software: quais remediações são seguras para automatizar, quando escalar versus esperar, como evitar a fadiga de alertas que treina as pessoas a ignorar o pager, e como capturar uma linha do tempo boa o bastante para que o post-mortem produza correções reais em vez de culpa.

## Pré-requisitos

- Uma stack de observabilidade que produz alertas em sinais de SLO ou saúde
- Uma plataforma de notificação/chat para integrar (ferramenta de paging, canais de chat)
- Entendimento de conceitos de on-call, escalonamento e severidade
- Familiaridade com runbooks e a ideia de ações seguras e reversíveis

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Rotear alertas ao respondente certo com base em serviço e severidade
- Automatizar a criação de incidentes: canal, linha do tempo e papéis
- Rodar remediações de primeira linha seguras e reversíveis automaticamente com salvaguardas
- Projetar escalonamento que alcança um humano quando a automação não basta
- Capturar uma linha do tempo e conduzir um post-mortem sem culpa com itens de ação rastreados

## Requisitos Funcionais

1. Um alerta deve ser classificado por severidade e roteado ao respondente on-call correto.
2. Declarar um incidente deve criar automaticamente um canal, uma linha do tempo e atribuir papéis.
3. Remediações seguras definidas devem rodar automaticamente, com salvaguarda e registro de auditoria.
4. Se a automação não resolver dentro de um limiar, o sistema deve escalar.
5. Toda ação humana e automatizada deve ser anexada a uma linha do tempo do incidente.
6. A resolução deve disparar um template de post-mortem pré-preenchido a partir da linha do tempo.
7. Itens de ação do post-mortem devem ser rastreados até a conclusão.

## Marcos Sugeridos

1. **Marco 1 — Detectar e rotear:** Classifique alertas por severidade e acione o on-call correto.
2. **Marco 2 — Orquestração de incidentes:** Auto-crie o canal, a linha do tempo e as atribuições de papel na declaração.
3. **Marco 3 — Auto-remediação segura:** Rode uma remediação reversível com salvaguarda e escalonamento de fallback.
4. **Marco 4 — Aprender:** Gere um post-mortem a partir da linha do tempo e leve os itens de ação até o encerramento.

## Esboço de Dados e Interface

```text
   alerta dispara
      │  classifica severidade (SEV1..SEV3), mapeia serviço -> on-call
      ▼
   ┌────────────────┐   aciona    ┌──────────────┐
   │ Orquestrador    │────────────▶│  On-call     │
   │ de incidente    │             └──────────────┘
   └───┬───────┬─────┘
       │       │ cria canal + linha do tempo + papéis (IC, comms, ops)
       │       ▼
       │  ┌──────────────┐
       │  │ Linha do tempo│  anexa cada ação, com timestamp
       │  │ do incidente  │
       │  └──────┬────────┘
       │ remediação segura? (salvaguarda: reversível, com escopo)
       ▼   sim -> roda + registra ; não/timeout -> escala
   ┌────────────────┐   resolve   ┌──────────────┐
   │ Auto-remediar  │────────────▶│ Post-mortem  │ (sem culpa, itens de ação)
   └────────────────┘             └──────────────┘

Destinatários de notificação usam identidades de exemplo, ex.: oncall@example.com

Metas não-funcionais:
  MTTD   tempo-para-detectar medido e em queda
  MTTR   tempo-para-resolver medido por severidade
  precisão de alerta  razão de alertas acionáveis monitorada (combate à fadiga)
  segurança da automação só ações reversíveis e com escopo rodam sem supervisão
```

## Desafios Extras

- Adicione dicas de causa raiz auto-detectadas correlacionando o alerta com deploys recentes e traces.
- Introduza automação de comunicação por severidade (atualizações de status page, notificações a stakeholders).
- Adicione um modo "incidente de treino" para ensaiar o fluxo sem uma queda real.
- Rastreie a queima do error budget e auto-declare incidentes quando a taxa de queima for crítica.

## Definição de Pronto

- [ ] Alertas são classificados e roteados ao respondente correto por serviço e severidade.
- [ ] Declarar um incidente auto-cria um canal, linha do tempo e atribuições de papel.
- [ ] Pelo menos uma remediação segura e reversível roda automaticamente com salvaguarda.
- [ ] O escalonamento alcança um humano quando a automação falha ou expira.
- [ ] Um post-mortem é gerado a partir da linha do tempo e seus itens de ação são rastreados.

## Armadilhas Comuns

- Automatizar uma remediação que não é reversível, transformando um incidente pequeno em um grande.
- Fadiga de alertas: acionar por tudo até que os respondentes silenciem o canal que importa.
- Sem disciplina de linha do tempo, o post-mortem vira adivinhação e não produz correções reais.
- Escalonamento sem timeout, então um incidente não resolvido fica com a automação para sempre.
- Post-mortems com culpa que fazem as pessoas esconderem detalhes, derrotando todo o propósito.

## Recursos

- [Google SRE Book: Managing Incidents](https://sre.google/sre-book/managing-incidents/) — comando e coordenação de incidentes.
- [Google SRE Book: Postmortem Culture](https://sre.google/sre-book/postmortem-culture/) — aprendizado sem culpa bem feito.
- [PagerDuty Incident Response](https://response.pagerduty.com/) — um guia prático e aberto de resposta a incidentes.
- [Atlassian: Incident management](https://www.atlassian.com/incident-management) — fundamentos de severidade, papéis e processo.
