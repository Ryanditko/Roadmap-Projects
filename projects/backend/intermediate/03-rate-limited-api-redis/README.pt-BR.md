# Rate-Limited API with Redis

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Toda API pública acaba precisando de um segurança na porta — algo que deixa o tráfego normal passar, mas impede que um cliente barulhento afogue o serviço. Neste projeto você constrói esse segurança como um middleware reutilizável de rate limiting apoiado no Redis. O detalhe interessante é que sua API pode rodar em várias instâncias atrás de um load balancer, então contar requisições na memória local deixaria o cliente multiplicar sua cota pelo número de servidores. O Redis vira a fonte única de verdade compartilhada por todas as instâncias, e você compara algoritmos reais — janela fixa, janela deslizante, token bucket — em vez de recorrer a uma biblioteca que esconde os trade-offs.

## Pré-requisitos

- Conforto para construir endpoints HTTP e middlewares ([API REST Simples de Tarefas](../../beginner/01-simple-rest-api-task-management/) é uma boa base)
- Uma instância Redis rodando (um container Docker local serve)
- Entender códigos de status HTTP, especialmente o `429 Too Many Requests`
- Familiaridade com operações atômicas e por que elas importam sob concorrência

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Comparar os algoritmos de janela fixa, janela deslizante e token bucket e seus trade-offs de memória/precisão
- Usar comandos atômicos do Redis (`INCR`, `EXPIRE`, sorted sets ou um script Lua) para contar com segurança entre instâncias
- Retornar os cabeçalhos padrão de rate limit para que os clientes se autolimitem
- Distinguir limitação por usuário (autenticado) de limitação por IP (anônimo)
- Raciocinar sobre o que acontece quando o Redis está lento ou indisponível

## Requisitos Funcionais

1. O sistema deve impor um limite configurável de N requisições por janela de tempo em cada endpoint protegido.
2. Requisições acima do limite devem ser rejeitadas com HTTP `429` e um cabeçalho `Retry-After`.
3. A contagem deve ser compartilhada entre várias instâncias da API via Redis, não mantida em memória local.
4. O incremento do contador e sua expiração devem ser atômicos para que requisições concorrentes não driblem o limite.
5. Toda resposta deve incluir os cabeçalhos `RateLimit-Limit`, `RateLimit-Remaining` e `RateLimit-Reset`.
6. Requisições autenticadas devem ser limitadas pela identidade do usuário; requisições anônimas pelo IP do cliente.
7. Uma allowlist configurada de identidades ou IPs deve ignorar a limitação por completo.

## Marcos Sugeridos

1. **Marco 1 — Janela fixa:** Implemente `INCR` + `EXPIRE` por chave/janela, retornando 429 ao passar do limite.
2. **Marco 2 — Algoritmo melhor:** Substitua a janela fixa por uma janela deslizante (sorted set) ou token bucket para suavizar o comportamento de rajada nas bordas da janela.
3. **Marco 3 — Cabeçalhos, identidades e allowlist:** Emita os cabeçalhos padrão, separe as chaves por usuário e por IP e respeite uma allowlist.

## Esboço de Dados e Interface

```text
Chaves Redis
  ratelimit:user:{id}:{window}   -> contador (janela fixa)
  ratelimit:ip:{addr}            -> sorted set de timestamps de requisição (deslizante)
  tokenbucket:{id}               -> hash { tokens, lastRefill }

Cabeçalhos de resposta (permitidos)
  RateLimit-Limit: 100
  RateLimit-Remaining: 87
  RateLimit-Reset: 1710000000

Resposta quando bloqueado
  HTTP 429 Too Many Requests
  Retry-After: 15
  corpo: { "error": "rate_limit_exceeded", "retryAfter": 15 }

Escolhas de algoritmo: janela fixa | log de janela deslizante | token bucket
```

## Desafios Extras

- Adicione limites por camada (plano grátis vs. premium) resolvidos a partir da identidade autenticada.
- Implemente uma tolerância de rajada sobre uma taxa de reabastecimento constante com o modelo token bucket.
- Adicione um endpoint administrativo para inspecionar e ajustar limites em tempo de execução.
- Mova a lógica de checar-e-incrementar para um único script Lua no Redis, eliminando as idas e voltas.

## Definição de Pronto

- [ ] Um cliente que excede o limite recebe consistentemente 429 com um `Retry-After` correto.
- [ ] Rodar duas instâncias da API contra um único Redis impõe um limite compartilhado único.
- [ ] Requisições concorrentes perto da fronteira nunca ultrapassam a contagem configurada.
- [ ] Os cabeçalhos padrão de rate limit aparecem tanto nas respostas permitidas quanto nas bloqueadas.
- [ ] Identidades na allowlist nunca são limitadas.

## Armadilhas Comuns

- Usar `GET` e depois `SET` separados em vez de um `INCR` atômico — uma corrida deixa requisições escaparem.
- Esquecer de definir uma expiração no primeiro incremento, então os contadores vivem para sempre e nunca reiniciam.
- O problema de borda da janela fixa: um cliente pode enviar 2×N requisições na virada entre duas janelas.
- Bloquear toda requisição em uma chamada síncrona ao Redis sem timeout, de modo que um Redis lento trava a API inteira.
- Chavear limites anônimos por um IP atrás de um proxy sem ler o `X-Forwarded-For`, agrupando todos os usuários juntos.

## Recursos

- [Redis: comando INCR](https://redis.io/docs/latest/commands/incr/) — o padrão canônico de contador atômico, com exemplo de rate limiter.
- [Redis: padrões de rate limiting](https://redis.io/glossary/rate-limiting/) — janela deslizante e token bucket explicados.
- [MDN: 429 Too Many Requests](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status/429) — semântica do status e `Retry-After`.
- [Draft IETF: cabeçalhos RateLimit](https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/) — os cabeçalhos padrão que sua API deve emitir.
- [Cloudflare: como funciona o rate limiting](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/) — uma visão geral dos conceitos em linguagem simples.
