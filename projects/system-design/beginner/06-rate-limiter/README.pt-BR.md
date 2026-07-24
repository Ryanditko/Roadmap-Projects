# Projete um Rate Limiter

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete um rate limiter que limita quantas requisições um cliente pode fazer em uma janela — o componente que retorna `429 Too Many Requests` quando alguém martela sua API. A questão central é qual algoritmo usar (token bucket, sliding window, fixed window) e onde o contador vive. Em um único servidor, contar é trivial; em uma frota atrás de um load balancer, o contador precisa ser compartilhado, o que traz um store central rápido e suas dores de consistência. Entregue um documento de design cobrindo o algoritmo, o store do contador e o contrato com o cliente.

## Pré-requisitos

- Entendimento do que são uma requisição de API e uma janela de tempo
- Consciência de que vários servidores não podem cada um manter sua própria contagem privada
- Familiaridade com um store chave-valor rápido como o Redis em nível conceitual
- Conforto para raciocinar sobre contadores e expiração

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Comparar algoritmos de rate limiting e seu comportamento em rajadas
- Decidir onde o contador vive em uma implantação distribuída
- Projetar o contrato HTTP para uma requisição rejeitada (status e cabeçalhos)
- Raciocinar sobre precisão vs. desempenho em um contador compartilhado
- Enunciar um trade-off entre precisão estrita e baixa latência

## Requisitos e Restrições

1. Limitar cada cliente (por API key ou IP) a N requisições por janela de tempo.
2. Rejeitar requisições acima do limite com `429` e cabeçalhos dizendo ao cliente quando tentar de novo.
3. Funcionar corretamente entre muitos servidores de aplicação, não só um.
4. Adicionar latência mínima às requisições permitidas (overhead alvo < 5 ms).
5. Suportar limites diferentes por endpoint ou por tier de cliente.
6. Estime a escala: 50K requisições/s no pico em toda a frota.

## Abordagem Sugerida

1. Escolha um algoritmo e raciocine sobre seu comportamento de borda — fixed window tem rajadas na fronteira; sliding window e token bucket as suavizam.
2. Faça a conta: a 50K req/s, o store do contador precisa aguentar ao menos essa quantidade de operações read-modify-write — planeje para isso.
3. Decida a localização do contador: local (rápido, impreciso entre nós) vs. store central (preciso, adiciona um hop).
4. Defina o contrato de resposta: `429` mais cabeçalhos `Retry-After` e `X-RateLimit-*`.
5. Descreva como os contadores expiram e como limites por tier são configurados.

## Esboço de Arquitetura

```text
Cliente ──> [ LB ] ──> [ Nó App ] ── check+incr ──> [ Store Central de Contador ]
                            │                              (key: client:window)
                      allow │  deny
                            ▼    ▼
                          passa  429 + Retry-After

Algoritmos
  Fixed window     contagem por bucket fixo          (simples, rajadas na fronteira)
  Sliding window   ponderado entre dois buckets       (mais suave, mais computação)
  Token bucket     recarrega tokens a taxa constante   (permite rajadas controladas)

Modelo de dados (store central)
  key: "{clientId}:{window}"  value: count      TTL = tamanho da janela
  config: client_id/tier -> limit, window
```

## Tópicos de Aprofundamento

- **Escolha do algoritmo:** por que fixed-window permite uma rajada de 2× nas fronteiras, e como token bucket permite rajadas intencionais.
- **Contagem distribuída:** incremento atômico no store central vs. contadores locais aproximados sincronizados periodicamente.
- **Modo de falha:** fail-open (permitir na queda do store) vs. fail-closed (rejeitar) e o risco de cada um.

## Entregáveis

- Um diagrama de arquitetura mostrando o nó de app, o store do contador e o caminho de decisão.
- O contrato com o cliente: código de status e cabeçalhos de rate limit.
- Um modelo de dados para as chaves de contador e a config por tier.
- Um trade-off descrito: ex., store central (limites globais precisos, latência e dependência extra) vs. contadores locais (rápidos, mas os limites são só aproximados entre nós).

## Armadilhas Comuns

- Manter contagens locais em cada servidor, de modo que o limite real vira N × (número de servidores).
- Usar uma fixed window e ignorar a rajada dupla nas fronteiras da janela.
- Retornar `429` sem `Retry-After`, deixando os clientes adivinharem e repetirem agressivamente.
- Não decidir fail-open vs. fail-closed, deixando o comportamento durante uma queda do store indefinido.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — padrões de sistemas distribuídos e cache.
- [Cloudflare: How we built rate limiting](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/) — uma abordagem de sliding-window em produção.
- [Stripe: Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters) — algoritmos e trade-offs práticos.
- [MDN: 429 Too Many Requests](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status/429) — a resposta e o cabeçalho `Retry-After`.
