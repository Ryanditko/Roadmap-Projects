# Projete um API Gateway

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete um API gateway que fica à frente de uma frota de microsserviços: é o ponto único de entrada que autentica chamadores, roteia cada requisição ao backend certo, aplica limites de taxa e protege os serviços de picos de tráfego. Toda requisição do sistema passa por ele, então ele precisa ser rápido, altamente disponível e stateless o bastante para escalar horizontalmente. As perguntas de design são onde manter estado compartilhado (tokens de auth, contadores de rate limit), como descobrir backends e como falhar graciosamente. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento de proxies reversos e balanceamento de carga
- Familiaridade com auth baseada em token (JWT/OAuth) em nível conceitual
- Noção de algoritmos de rate limiting (token bucket, janela deslizante)
- Conforto para estimar throughput de requisições e orçamento de latência por requisição

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar o pipeline de requisição: auth → rate limit → rotear → transformar → encaminhar
- Estimar o QPS do gateway e o orçamento de latência adicionado por estágio
- Projetar um rate limiter distribuído com contadores compartilhados
- Projetar service discovery e health checking para backends dinâmicos
- Justificar trade-offs entre estado centralizado e por instância

## Requisitos e Restrições

- Assuma 100k requisições/s entre 50 serviços de backend, latência adicionada p99 abaixo de ~10 ms.
- O gateway deve permanecer disponível mesmo se alguns backends caírem (circuit breaking).
- Limites de taxa são aplicados por API key em toda a frota do gateway, não por instância.
- A auth deve validar tokens sem um round-trip por requisição sempre que possível.
- Estime o QPS do gateway, o armazenamento de contadores de rate limit e o tamanho do cache de auth/rotas.

## Abordagem Sugerida

1. Defina os estágios do pipeline de requisição e o orçamento de latência de cada um.
2. Decida onde a validação de auth ocorre: verificação local de JWT vs. chamar um serviço de auth.
3. Projete rate limiting distribuído: contadores compartilhados no Redis vs. contadores locais aproximados.
4. Projete service discovery (registro + health checks) e balanceamento de carga.
5. Adicione resiliência: circuit breakers, timeouts, retentativas com backoff.

## Esboço de Arquitetura

```text
Clientes -> [LB] -> Frota do gateway (stateless) --pipeline--> backends
                       |  1. auth (verifica JWT localmente; cacheia JWKS)
                       |  2. rate limit (token bucket no Redis por apiKey)
                       |  3. rotear (path -> serviço via registro)
                       |  4. transformar (headers, versionamento)
                       |  5. encaminhar (+ circuit breaker, timeout, retry)
        Registro de serviços <- health checks -> instâncias de backend

ANY /v1/{service}/*   -> encaminha ao backend resolvido
GET /_health          -> 200 saúde do gateway

Route      { pathPrefix, serviceId, version }              // cacheado em memória, atualizado
RateLimit  { apiKey -> tokens, refillTs }                  // Redis, particiona por apiKey
Backend    { serviceId, instances[{host, healthy}], lbPolicy }
```

## Tópicos de Aprofundamento

- **Rate limiting:** token bucket vs. janela deslizante; consistência do contador compartilhado vs. velocidade.
- **Resiliência:** estados do circuit breaker (fechado/aberto/meio-aberto), orçamentos de timeout e retry.
- **Trade-off 1 — estado de rate limit centralizado vs. local:** um contador compartilhado no Redis é exato mas adiciona um salto de rede e uma dependência; contadores locais são rápidos mas só aproximam o limite global. Justifique contadores locais com sincronização periódica para alto QPS.
- **Trade-off 2 — estratégia de auth:** verificação local de JWT evita um round-trip por requisição mas torna a revogação de token lenta; uma chamada central de auth é autoritativa mas adiciona latência. Justifique JWTs de vida curta mais um cache de revogação.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo o pipeline e a arquitetura acima.
- [ ] Estimativas de capacidade: QPS do gateway, orçamento de latência por estágio, armazenamento de rate limit + cache de rotas.
- [ ] Um plano de particionamento dos contadores de rate limit por API key.
- [ ] Uma estratégia de cache para JWKS, rotas e decisões de auth com política de atualização.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Tornar o gateway stateful, impedindo que instâncias escalem ou façam failover de forma limpa.
- Aplicar limites de taxa por instância, deixando o tráfego total exceder o limite global pretendido.
- Sem circuit breaker, fazendo um backend lento esgotar as threads do gateway e derrubar tudo.
- Validar tokens chamando o serviço de auth a cada requisição, estourando o orçamento de latência.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — proxies, balanceamento de carga e confiabilidade.
- [NGINX: o que é um API gateway](https://www.nginx.com/learn/api-gateway/) — responsabilidades do gateway na prática.
- [Martin Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) — o padrão de resiliência explicado.
- [Stripe: scaling your API with rate limiters](https://stripe.com/blog/rate-limiters) — rate limiting distribuído no mundo real.
