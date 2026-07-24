# Projete uma CDN

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete uma Content Delivery Network — a frota de servidores de borda que fica entre os usuários e a infraestrutura de origem para servir conteúdo estático e transmissível (imagens, bundles JS/CSS, segmentos de vídeo) a partir de um local fisicamente próximo de cada visitante. A tensão interessante do design não é "coloque um cache perto do usuário", mas tudo que vem depois: como rotear uma requisição para a borda certa, como manter as cópias em cache frescas sem sobrecarregar a origem e como purgar um asset ruim de centenas de pontos de presença em segundos. Este é um exercício de design — o entregável é um documento de design escrito com diagramas, não uma CDN em execução.

## Pré-requisitos

- Conforto com a semântica de cache HTTP ([Projete um Rate Limiter](../../beginner/06-rate-limiter/) ou um projeto de caching intermediário é um bom aquecimento)
- Entendimento de resolução DNS e de como clientes escolhem um servidor
- Familiaridade com cabeçalhos de cache: `Cache-Control`, `ETag`, `Last-Modified`, TTL
- Estimativa básica de ordem de grandeza (requisições/seg, banda, armazenamento)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Estimar a capacidade da borda a partir de tráfego, tamanho de objeto e taxa de acerto de cache, e justificar os números
- Comparar estratégias de roteamento de requisições (baseada em DNS, Anycast, GeoDNS) e escolher uma com razões
- Projetar uma estratégia de frescor e invalidação de cache, incluindo purga quase instantânea
- Particionar o namespace de objetos entre nós de borda (hashing consistente) e explicar o rebalanceamento
- Articular ao menos duas decisões de trade-off com a alternativa rejeitada e o porquê

## Requisitos e Restrições

**Funcionais**

- Servir conteúdo cacheável a partir da borda mais próxima do usuário, recorrendo à origem em um miss.
- Suportar invalidação de cache: purgar um objeto específico ou um prefixo de caminho globalmente em segundos.
- Respeitar as diretivas de cache da origem (TTL, `no-store`, revalidação por `ETag`).
- Expor uma API para o cliente configurar origens, regras de cache e disparar purgas.

**Não-funcionais**

- Alvo de taxa de acerto de cache ≥ 90% para assets estáticos; resposta de borda p99 < 50 ms em um hit.
- Disponibilidade 99,9%+; a falha de uma borda não pode derrubar a entrega de conteúdo de sua região.
- Proteger a origem de thundering-herd em cache frio ou invalidação em massa.
- Consciente de custo: minimizar egress da origem e banda entre PoPs.

## Abordagem Sugerida

1. **Estime primeiro.** Escolha um cenário (ex.: 50 TB/dia de egress, objeto médio de 5 KB, 90% de acerto). Derive requisições/seg de pico, banda por PoP e armazenamento do hot-set. Deixe esses números guiarem a contagem de PoPs e o dimensionamento do cache.
2. **Escolha o roteamento de requisições.** Compare GeoDNS vs. Anycast para direcionar usuários a um PoP. Observe os efeitos de cache do TTL de DNS e a latência de failover.
3. **Projete a camada de cache.** Decida a política de despejo (LRU vs. LFU vs. TinyLFU) e um layout de duas camadas (borda → shield regional → origem) para absorver misses.
4. **Projete a invalidação.** Escolha entre TTL curto, URLs versionadas e purga ativa. Esboce como uma purga se propaga para todos os PoPs.
5. **Particione o namespace.** Use hashing consistente para que objetos mapeiem para nós com o mínimo de remanejamento quando um nó entra ou sai.

## Esboço de Arquitetura

```text
                 ┌────────── Plano de controle ─────────┐
   Cliente ────► │  API de Config · API de Purga · Métricas │
                 └───────────────────┬──────────────────┘
                                     │ (envia config / purga)
   Usuário ─DNS/Anycast─► PoP (borda) ─miss─► Shield regional ─miss─► Origem
        (mais próximo)     │  cache LRU/TinyLFU       │  cache maior
                          └─hit─► resposta            └─hit─► resposta

Objeto (entrada de cache)
  key:        hash(host + caminho + variante)
  bytes:      blob
  ttl:        segundos     (do Cache-Control da origem)
  etag:       string       (para revalidação)
  storedAt:   epoch ms

API do Cliente
  PUT  /v1/distributions/{id}      body: { origin, cacheRules[] }
  POST /v1/distributions/{id}/purge body: { paths: ["/img/*"] } -> 202 { purgeId }
  GET  /v1/distributions/{id}/stats -> { hitRatio, egressBytes, p99Ms }

Roteamento: usuário -> GeoDNS resolve para o VIP do PoP mais próximo
Particionamento: anel de hash consistente mapeia key do objeto -> nó de borda
```

## Tópicos de Aprofundamento

- **Hierarquia de cache e shields:** Por que uma camada intermediária de shield reduz a carga na origem e como ela muda a taxa de acerto efetiva.
- **Semântica de invalidação:** Fan-out de purga por caminho vs. cache-busting via URLs versionadas — trade-offs de latência e correção.
- **Hashing consistente:** Nós virtuais para carga uniforme; comportamento quando um PoP é adicionado ou falha.
- **Prevenção de cache stampede:** Coalescência de requisições / single-flight e stale-while-revalidate.
- **Segurança na borda:** Terminação TLS, URLs assinadas e absorção básica de DDoS.

## Entregáveis

- Um documento de design (~2–4 páginas) cobrindo roteamento, caching, particionamento e invalidação.
- Estimativa de capacidade: RPS de pico, banda por PoP e armazenamento do hot-set, com premissas declaradas.
- O diagrama de arquitetura, o modelo de dados e o contrato da API do cliente.
- Ao menos dois trade-offs justificados (ex.: GeoDNS vs. Anycast; TTL curto vs. purga ativa), nomeando a opção rejeitada e o porquê.

## Armadilhas Comuns

- Reportar uma taxa de acerto sem estimar o tamanho do hot-set que a torna alcançável.
- Escolher roteamento baseado em DNS sem considerar o cache de TTL do resolver, que atrasa o failover.
- Tratar a invalidação como algo secundário — a latência de purga global é uma métrica central de CDN, não uma nota de rodapé.
- Ignorar o thundering-herd em cache frio; sem coalescência, um miss popular pode sobrecarregar a origem.
- Escolher hashing por módulo para particionamento, que remaneja quase tudo quando a contagem de nós muda.

## Recursos

- [Cloudflare: O que é uma CDN?](https://www.cloudflare.com/pt-br/learning/cdn/what-is-a-cdn/) — introdução clara sobre entrega de borda e PoPs.
- [MDN: Cache HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Caching) — semântica de `Cache-Control`, `ETag` e revalidação.
- [System Design Primer: CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network) — trade-offs de CDNs push vs. pull.
- [Hashing Consistente (artigo original)](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf) — a base para o particionamento objeto-para-nó.
- [web.dev: Cache HTTP](https://web.dev/articles/http-cache) — orientação prática sobre estratégia de caching.
