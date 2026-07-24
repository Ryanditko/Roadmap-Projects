# Pipelines Autorrecuperáveis

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Engineering · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa pipelines de dados que detectam suas próprias falhas e se recuperam automaticamente — retentando erros transitórios, colocando registros venenosos em quarentena, redirecionando ao redor de uma dependência morta e revertendo uma execução ruim — para que um humano seja acionado por novidade genuína, não pela mesma falha instável às 3 da manhã. O desafio de design é fazer isso *com segurança*: uma auto-remediação afoita demais pode amplificar um incidente (tempestades de retry martelando um downstream em dificuldade, um rollback que apaga dados bons). Você classificará falhas (transitória vs permanente), escolherá a resposta certa por classe (retry com backoff, circuit-break, quarentena, compensar/reverter), e adicionará health checks e detecção de anomalias para dispará-las. O tema é degradação graciosa com guardrails. A entrega é um pipeline que sobrevive a falhas injetadas com comportamento de recuperação documentado e limites de raio de impacto.

## Pré-requisitos

- Experiência construindo pipelines e um orquestrador (Airflow, Dagster, Temporal ou similar)
- Entendimento de estratégias de retry, idempotência e backoff exponencial
- Familiaridade com circuit breakers, dead-letter queues e health checks
- Conforto para raciocinar sobre modos de falha e raio de impacto

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Classificar falhas como transitórias vs permanentes e responder apropriadamente a cada uma
- Implementar retries seguros com backoff exponencial e jitter, limitados por idempotência
- Usar circuit breakers e dead-letter/quarentena para conter uma dependência falha ou registros ruins
- Projetar compensação/rollback para que uma falha parcial deixe estado consistente
- Detectar anomalias (volume, latência, taxa de erro) que disparam remediação antes de uma falha dura

## Requisitos Funcionais

1. O pipeline deve distinguir falhas transitórias (retentáveis) das permanentes (não) e agir diferente em cada uma.
2. Falhas transitórias devem retentar com backoff exponencial e jitter, com teto, e apenas onde as operações são idempotentes.
3. Registros que falham repetidamente devem ir para quarentena em um dead-letter store, não bloquear o pipeline inteiro.
4. Uma dependência a jusante falha deve acionar um circuit breaker que para de martelá-la e recupera quando ela se cura.
5. Uma execução falha deve reverter ou compensar para que nenhuma saída parcial e inconsistente seja publicada.
6. Health checks e ao menos um sinal de anomalia devem conseguir disparar remediação automática, com toda ação registrada.

## Marcos Sugeridos

1. **Marco 1 — Retry e classificar:** Adicione classificação de falhas e retry-com-backoff idempotente; injete erros transitórios e observe a recuperação.
2. **Marco 2 — Conter:** Adicione um caminho de dead-letter/quarentena e um circuit breaker para uma dependência instável.
3. **Marco 3 — Rollback e detectar:** Adicione compensação/rollback para uma execução ruim e um detector de anomalias que dispara remediação, tudo auditado.

## Esboço de Dados e Interface

```text
                 ┌─────────── classificador de falhas ───────────┐
   [estágio] ─err─▶ transitória? ──sim──▶ retry(backoff, jitter, maxN)  (só idempotente)
                 └ permanente? ──sim──▶ quarentena do registro ─▶ [dead-letter store]
                                         (o pipeline segue fluindo)

chamada a dependência ─▶ [circuit breaker]  fechado -> permite | aberto -> falha rápido + pula
                          aciona em taxa_erro > limite; meio-aberto sonda para recuperar

resultado da execução:
  sucesso -> publicação atômica
  falha   -> compensar/reverter -> nenhuma saída parcial visível

detector de anomalia: observa {volume_entrada, latência, taxa_erro}
   desvio > k*desvio_padrão -> dispara remediação + ACIONA se não reconhecido

guardrails: máximo de retries, cooldown do breaker, rate-limit de remediação
log de auditoria: toda ação automática { tempo, gatilho, ação, resultado }
```

## Desafios Extras

- Adicione uma camada de "aprendizado" que rastreia assinaturas de falha recorrentes e sugere (ou aplica) uma correção conhecida.
- Implemente backfill automático de um lote em quarentena assim que a causa raiz for resolvida.
- Adicione teste de caos que injeta falhas aleatoriamente no CI para provar que a recuperação de fato funciona.

## Definição de Pronto

- [ ] Falhas transitórias e permanentes são classificadas e tratadas de forma diferente.
- [ ] Retries idempotentes com backoff+jitter limitados recuperam de erros transitórios injetados.
- [ ] Registros venenosos caem em um dead-letter store sem travar o pipeline.
- [ ] Um circuit breaker protege uma dependência instável e recupera automaticamente.
- [ ] Uma execução falha não deixa saída parcial; toda remediação automática é registrada e limitada por taxa.

## Armadilhas Comuns

- Tempestades de retry: retries ilimitados ou sem jitter transformam um soluço em uma queda autoinfligida.
- Retentar operações não idempotentes e aplicar efeitos colaterais em dobro.
- Auto-recuperar tão agressivamente que mascara um bug real — ninguém nunca descobre que o pipeline está quebrado.
- Rollback que apaga ou corrompe dados bons porque a compensação não foi escopada à execução falha.

## Recursos

- [Livro SRE do Google: Lidando com Sobrecarga e Falhas em Cascata](https://sre.google/sre-book/handling-overload/) — orçamentos de retry e load shedding bem feitos.
- [AWS: Timeouts, retries e backoff com jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) — a orientação canônica de retry seguro.
- [Martin Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) — o padrão para conter uma dependência falha.
- [Airflow: Retries e callbacks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html) — retry e hooks de falha no nível do orquestrador.
