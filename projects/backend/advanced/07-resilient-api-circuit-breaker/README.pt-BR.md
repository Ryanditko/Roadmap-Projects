# API Resiliente com Circuit Breaker

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa uma API que continua útil quando suas dependências não continuam. Sistemas reais chamam outros sistemas — um gateway de pagamento, um serviço de recomendação, um banco de dados — e qualquer um deles pode ficar lento ou falhar. Sem proteção, uma dependência lenta prende toda thread de requisição e derruba o serviço inteiro junto. Este projeto é sobre os padrões de resiliência que estancam essa cascata: o **circuit breaker** que para de martelar uma dependência que falha, o **bulkhead** que isola as falhas de uma dependência das demais, **timeouts** que limitam quanto tempo você espera, **fallbacks** que retornam algo sensato em vez de um erro e o **load shedding** que protege o serviço sob sobrecarga. O objetivo é a degradação graciosa — um serviço que fica mais estreito sob estresse em vez de tombar.

## Pré-requisitos

- Experiência sólida construindo APIs HTTP e chamando serviços downstream
- Entendimento de threads, pools de conexão e I/O bloqueante vs. assíncrono
- Conforto com primitivas de concorrência (thread pools, semáforos, timeouts)
- Familiaridade com conceitos básicos de métricas/monitoramento
- Uma stack de backend de sua escolha (Java/Kotlin, Go, Node, Python, etc.)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar um circuit breaker como uma máquina de estados (fechado → aberto → meio-aberto)
- Isolar dependências com bulkheads para que uma falha não esgote recursos compartilhados
- Escolher e impor timeouts sensatos, e explicar por que "sem timeout" é um bug
- Projetar fallbacks que degradam a funcionalidade graciosamente em vez de dar erro
- Descartar carga (shed) sob sobrecarga para proteger a latência das requisições que você atende
- Raciocinar sobre os trade-offs de cada padrão (atualidade, disparos falsos, ajuste)

## Requisitos Funcionais

1. O serviço deve envolver chamadas a uma dependência downstream não confiável atrás de um circuit breaker com três estados: **fechado** (chamadas passam), **aberto** (chamadas falham rápido) e **meio-aberto** (uma sondagem limitada de chamadas testa a recuperação).
2. O breaker deve abrir quando um limiar de falha configurável é cruzado (taxa de erro ou falhas consecutivas) e, após um cooldown, mover para meio-aberto para testar a dependência.
3. Toda chamada downstream deve ter um timeout; uma chamada que o exceder conta como falha, nunca uma espera indefinida.
4. Cada dependência downstream deve ser isolada por um bulkhead (um thread pool limitado ou semáforo) para que saturar uma não possa faminta as outras.
5. Quando o breaker está aberto ou uma chamada falha, o serviço deve retornar um fallback definido (valor cacheado, padrão ou resposta parcial) em vez de propagar um erro cru sempre que possível.
6. O serviço deve descartar carga quando passar de um limite de concorrência ou profundidade de fila, rejeitando requisições em excesso rapidamente com 503 em vez de enfileirá-las indefinidamente.
7. **Não funcional:** sob uma dependência totalmente fora, a latência p99 do endpoint protegido deve permanecer limitada (fail-fast), e o serviço não deve esgotar suas próprias threads ou conexões. Ele deve permanecer disponível para endpoints que não dependem do serviço que falhou.
8. O serviço deve expor métricas (estado do breaker, contagens de sucesso/falha de chamadas, requisições rejeitadas/descartadas, latência) para observabilidade.

## Marcos Sugeridos

1. **Marco 1 — Timeouts e uma chamada envolvida:** Coloque uma dependência real (ou instável simulada) atrás de um cliente com timeout imposto e resultados estruturados de sucesso/falha.
2. **Marco 2 — Circuit breaker:** Implemente a máquina de estados fechado/aberto/meio-aberto com limiar e cooldown; verifique que ela falha rápido quando aberta.
3. **Marco 3 — Bulkhead e fallback:** Adicione isolamento por dependência e um caminho de fallback; prove que a falha de uma dependência não afeta outra.
4. **Marco 4 — Load shedding e métricas:** Adicione um limite de concorrência com rejeição rápida e exponha métricas de breaker/latência; rode um teste de carga para observar a degradação.

## Esboço de Dados e Interface

```text
Máquina de estados do Circuit Breaker

        falhas >= limiar
 FECHADO ─────────────────────────▶ ABERTO
   ▲                                   │
   │ sondagem tem sucesso              │ cooldown expirou
   │                                   ▼
   └────────── MEIO-ABERTO ◀───────────┘
        sondagem falha → volta a ABERTO

Caminho da requisição
 cliente ─▶ [load shedder] ─▶ [pool bulkhead] ─▶ [breaker] ─▶ downstream
                  │                                   │
                  └─ 503 se acima do limite           └─ fallback se aberto/falho

Config (por dependência)
  timeoutMs, failureThreshold, cooldownMs,
  bulkheadMaxConcurrent, halfOpenProbes

Métricas expostas
  breaker_state{dep}          gauge (0=fechado,1=aberto,2=meio_aberto)
  calls_total{dep,result}     counter
  requests_shed_total         counter
  call_latency_seconds        histogram
```

## Desafios Extras

- Adicione **timeouts adaptativos** baseados em percentis de latência observados em vez de um valor fixo.
- Implemente um **retry com backoff exponencial e jitter** na frente do breaker, e raciocine sobre como retries e breakers interagem.
- Adicione um pequeno **dashboard** ou endpoint de status visualizando os estados do breaker ao vivo.
- Suporte **load shedding baseado em prioridade**, mantendo requisições críticas e descartando as de baixa prioridade.

## Definição de Pronto

- [ ] O breaker demonstravelmente abre sob falhas sustentadas e se recupera via sondagens meio-abertas.
- [ ] Uma dependência morta causa falhas rápidas, não esgotamento de threads nem latência ilimitada.
- [ ] Bulkheads impedem que uma dependência saturada degrade endpoints não relacionados.
- [ ] Fallbacks retornam respostas utilizáveis onde definidos, com comportamento claro onde não.
- [ ] O load shedding rejeita tráfego em excesso rapidamente e mantém estável a latência das requisições atendidas.
- [ ] As métricas expõem estado do breaker, desfechos de chamadas e contagens de descarte.

## Armadilhas Comuns

- Fazer chamadas downstream sem timeout — a causa isolada mais comum de falha em cascata.
- Fazer retry agressivo na frente de um breaker aberto, amplificando a carga sobre uma dependência em dificuldade.
- Compartilhar um único thread pool entre todas as dependências, de modo que uma chamada lenta faminta tudo (sem bulkhead real).
- Definir o limiar de falha tão baixo que o breaker dispara em oscilações transitórias normais (falsos positivos).
- Fallbacks que servem silenciosamente dados obsoletos ou errados sem nenhum sinal de que houve degradação.
- Fazer load shedding deixando requisições enfileirarem até expirar de qualquer forma, em vez de rejeitar rápido.

## Recursos

- [Martin Fowler: CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html) — a explicação canônica do padrão.
- [Release It! (Michael Nygard)](https://pragprog.com/titles/mnee2/release-it-second-edition/) — o livro que popularizou circuit breakers, bulkheads e timeouts.
- [Documentação do Resilience4j](https://resilience4j.readme.io/docs/getting-started) — uma implementação de referência desses padrões para estudar.
- [AWS: Timeouts, retries e backoff com jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) — como retries e backoff interagem com resiliência.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — load shedding e degradação graciosa na prática.
