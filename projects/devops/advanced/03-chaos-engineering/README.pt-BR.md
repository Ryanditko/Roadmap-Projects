# Configuração de Chaos Engineering

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Quebre deliberadamente o seu próprio sistema — sob controle — para aprender como ele falha antes que seus usuários lhe ensinem do jeito difícil. Você vai construir uma prática de chaos engineering: formular uma hipótese ("se um pod morrer, o serviço permanece dentro do seu SLO de latência"), injetar uma falha real, medir o impacto em relação a um baseline de estado estável, e então confirmar a resiliência ou registrar a fraqueza encontrada. A disciplina que separa chaos engineering de "matar coisas aleatoriamente" é o método científico somado ao controle do raio de impacto: todo experimento deve ter escopo definido, uma condição de aborto automático e uma forma de parar instantaneamente. O objetivo não é caos por si só; é confiança, sustentada por evidências, de que seu sistema degrada de forma graciosa.

## Pré-requisitos

- Uma aplicação em execução com dependências relevantes (um datastore, um serviço a jusante) para perturbar
- Observabilidade instalada — métricas e idealmente traces — para medir o estado estável
- Conforto com Kubernetes ou as primitivas de falha da sua plataforma-alvo
- Entendimento de SLIs/SLOs para saber o que significa "ainda saudável"

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Definir hipóteses de estado estável em termos mensuráveis de SLI
- Injetar falhas (morte de pods, latência, partição de rede, exaustão de recursos) com segurança
- Impor limites de raio de impacto e condições de aborto automático
- Medir o impacto do experimento e distinguir sinal de ruído
- Transformar fraquezas descobertas em melhorias de resiliência rastreadas e corrigidas

## Requisitos Funcionais

1. Cada experimento deve declarar uma hipótese de estado estável em termos de um SLI mensurável.
2. O sistema deve injetar pelo menos três tipos distintos de falha (ex.: morte de pod, latência, partição).
3. Todo experimento deve ter um raio de impacto limitado (namespace, porcentagem ou seletor de label).
4. Um aborto automático deve interromper o experimento se uma métrica de guarda cruzar um limiar.
5. Deve existir um controle único e confiável de "pare tudo agora".
6. Resultados devem ser registrados: hipótese, falha, impacto observado, sucesso/falha, acompanhamento.
7. Experimentos devem ser repetíveis para que uma correção possa ser verificada contra a mesma falha.

## Marcos Sugeridos

1. **Marco 1 — Baseline e hipótese:** Estabeleça SLIs de estado estável e escreva sua primeira hipótese.
2. **Marco 2 — Primeira injeção:** Mate um pod dentro de um raio de impacto apertado, meça a recuperação, registre o resultado.
3. **Marco 3 — Salvaguardas:** Adicione aborto automático em violação de SLO e um interruptor global de parada.
4. **Marco 4 — Biblioteca de falhas e agenda:** Adicione falhas de latência, partição e recurso; rode experimentos com cadência e leve descobertas até o encerramento.

## Esboço de Dados e Interface

```text
   ┌──────────────┐   1. lê baseline      ┌───────────────┐
   │  Definição   │──────────────────────▶│ Observabilidade│
   │  do experim. │                       │ (SLIs/métricas)│
   └──────┬───────┘                       └──────┬────────┘
          │ 2. injeta falha                      │ 4. compara
          ▼   (raio de impacto limitado)         │    vs estado estável
   ┌──────────────┐                              ▼
   │ Motor de chaos│     violação de guarda? ┌───────────┐
   │ (Chaos Mesh/ │◀────── aborta/para ──────│ Veredito  │
   │  LitmusChaos)│                          │ ok/falha  │
   └──────────────┘                          └───────────┘

Registro do experimento:
  hipótese:    "latência p99 fica < 300ms se 1 de 3 pods morrer"
  falha:       pod-kill, seletor app=api, 33% das réplicas
  raio_impacto: namespace=staging, máx 1 pod
  aborta_se:   taxa_erro > 5% OU p99 > 600ms
  resultado:   ok | falha + ticket de acompanhamento

Metas não-funcionais:
  raio de impacto   nunca excede o escopo declarado
  latência de aborto < 10s da violação à parada
  MTTR observado    registrado para cada modo de falha testado
```

## Desafios Extras

- Evolua de staging para um game day controlado em produção com um raio de impacto pequeno.
- Adicione experimentos automatizados e agendados no CI para pegar regressões de resiliência.
- Injete falhas em nível de aplicação (erros de dependência, respostas lentas), não apenas falhas de infra.
- Correlacione resultados de experimentos com incidentes reais passados para priorizar quais falhas testar.

## Definição de Pronto

- [ ] Pelo menos três tipos de falha rodam com um raio de impacto declarado e imposto.
- [ ] Todo experimento tem um aborto automático e uma parada global funcional.
- [ ] Hipóteses de estado estável são medidas contra SLIs reais, não palpites.
- [ ] Fraquezas descobertas são rastreadas e ao menos uma foi corrigida e reverificada.
- [ ] Resultados dos experimentos são registrados em um formato repetível e revisável.

## Armadilhas Comuns

- Rodar chaos sem baseline de estado estável, sem saber se a falha importou.
- Nenhuma condição de aborto — um experimento "pequeno" vira uma queda real em cascata.
- Testar apenas mortes de pod, ignorando as falhas de latência e partição que causam incidentes reais.
- Tratar um experimento aprovado como permanente; sistemas mudam, então experimentos devem recorrer.
- Rodar em prod antes de as salvaguardas e a cultura estarem prontas, corroendo a confiança na prática.

## Recursos

- [Principles of Chaos Engineering](https://principlesofchaos.org/) — a definição e o método fundacionais.
- [Documentação do Chaos Mesh](https://chaos-mesh.org/docs/) — uma plataforma de chaos da CNCF para Kubernetes.
- [Documentação do LitmusChaos](https://docs.litmuschaos.io/) — framework de chaos engineering open-source.
- [Google SRE Book: Embracing Risk](https://sre.google/sre-book/embracing-risk/) — error budgets e raciocínio sobre falha.
