# API with Caching Layer (Redis)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Uma API que lê os mesmos dados raramente alterados do banco de dados a cada requisição faz muito trabalho caro sem motivo. Este projeto coloca um cache Redis na frente desse trabalho: a primeira requisição paga o custo completo e armazena o resultado, e as próximas mil requisições são respondidas a partir da memória em uma fração do tempo. A parte genuinamente interessante não é o caminho de leitura — é tudo que mantém o cache honesto. Quando um valor cacheado expira? O que acontece quando os dados subjacentes mudam? O que você faz quando mil requisições dão miss no mesmo instante? Fazer cache direito é, na maior parte, a disciplina de nunca servir dados nos quais você não pode mais confiar.

## Pré-requisitos

- Uma API existente com pelo menos um endpoint de leitura intensa apoiado em um datastore ([Plataforma de Blog com CRUD e Comentários](../02-blog-platform-crud/) é uma boa candidata)
- Uma instância Redis em execução e uma biblioteca cliente para sua linguagem
- Entendimento de TTL, serialização (JSON ou binária) e o Big-O básico das suas consultas
- Conforto para medir latência, de modo a provar que o cache realmente ajudou

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar o padrão cache-aside (lazy-loading) corretamente, incluindo o caminho do miss
- Escolher TTLs sensatos e raciocinar sobre o trade-off entre dado obsoleto e dado fresco
- Invalidar ou atualizar entradas do cache quando os dados de origem mudam
- Projetar um esquema de nomeação de chaves consistente e livre de colisões
- Detectar e mitigar o cache stampede (thundering herd) em uma chave quente
- Medir a razão de hit/miss e o uso de memória para saber se o cache vale a pena

## Requisitos Funcionais

1. Uma leitura cacheável deve checar o Redis primeiro e só ir ao banco de dados em caso de miss.
2. Em um miss, o valor buscado deve ser escrito de volta no cache com um TTL antes de ser retornado.
3. Uma escrita, atualização ou exclusão de um recurso deve invalidar (ou atualizar) sua representação cacheada, para que dado obsoleto nunca seja servido após uma mudança conhecida.
4. As chaves de cache devem seguir um esquema de nomeação documentado e livre de colisões, que codifique o recurso e quaisquer parâmetros.
5. O sistema deve expor estatísticas de cache — no mínimo contagem de hits, contagem de misses e razão de hits.
6. Uma indisponibilidade do Redis deve degradar graciosamente para o banco de dados, nunca retornando 500 puramente porque o cache está fora.
7. Misses concorrentes na mesma chave não devem todos estourar no banco de dados.

## Marcos Sugeridos

1. **Marco 1 — Cache read-through:** Implemente cache-aside em um endpoint com TTL e um esquema de chaves limpo; meça o ganho de latência.
2. **Marco 2 — Invalidação:** Conecte as escritas à invalidação ou atualização das chaves afetadas e confirme que nenhuma leitura obsoleta ocorre após uma mudança.
3. **Marco 3 — Resiliência e visibilidade:** Adicione proteção contra stampede, fallback gracioso na falha do Redis e um endpoint de estatísticas.

## Esboço de Dados e Interface

```text
Esquema de chave:  <recurso>:<versão>:<id>[:<hashParams>]
                   ex.: "post:v1:42"   "posts:v1:list:tag=redis&page=2"

Leitura cache-aside:
  value = redis.GET(key)
  if value == nil:                       # miss
      value = db.load(...)
      redis.SET(key, serialize(value), EX=ttl)
  return deserialize(value)

Invalidação na escrita:
  db.update(id, ...)
  redis.DEL("post:v1:42")                # ou SET com valor fresco
  redis.DEL(chaves de lista afetadas)    # incrementar versão para descartar muitas de uma vez

Proteção contra stampede: lock curto (SET NX) OU stale-while-revalidate OU TTL com jitter

GET  /posts/{id}      -> 200 (X-Cache: HIT|MISS)
GET  /admin/cache     -> 200 { hits, misses, hitRatio, keys, memoryMb }
```

## Desafios Extras

- Implemente stale-while-revalidate: sirva o valor expirado uma vez enquanto o atualiza em segundo plano.
- Adicione aquecimento de cache na inicialização para as chaves mais quentes.
- Use prefixos de chave versionados para invalidar uma classe inteira de entradas incrementando um contador de versão.
- Adicione configuração de TTL por endpoint e exponha o ajuste da política de eviction de memória do Redis (`maxmemory-policy`).

## Definição de Pronto

- [ ] Uma leitura repetida é mensuravelmente mais rápida na segunda chamada e reporta `X-Cache: HIT`.
- [ ] Atualizar um recurso reflete imediatamente na próxima leitura — nenhum valor obsoleto sobrevive a uma escrita conhecida.
- [ ] Derrubar o Redis no meio da execução ainda retorna respostas corretas a partir do banco de dados.
- [ ] O endpoint de estatísticas reporta uma razão de hits que sobe sob leituras repetidas.
- [ ] Uma rajada de misses concorrentes em uma chave fria dispara no máximo uma carga do banco de dados.

## Armadilhas Comuns

- Fazer cache sem um plano de invalidação, fazendo os usuários verem dados que mudaram minutos atrás.
- Esquecer de definir um TTL, deixando o cache crescer até o Redis fazer eviction de forma imprevisível.
- Cachear respostas de erro ou resultados vazios e depois servi-los muito depois de o dado existir (faça negative-cache com cuidado).
- Construir chaves a partir de entrada de usuário não sanitizada, causando colisões ou crescimento ilimitado de chaves.
- Tratar um timeout do Redis como fatal em vez de recorrer à fonte da verdade.

## Recursos

- [Redis: Padrões e boas práticas de caching](https://redis.io/docs/latest/develop/use/patterns/) — cache-aside, write-through e orientação de eviction direto da fonte.
- [AWS: Estratégias de caching (Lazy Loading vs Write-Through)](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/Strategies.html) — uma comparação clara com trade-offs.
- [Redis: Políticas de eviction de chaves](https://redis.io/docs/latest/develop/reference/eviction/) — como `maxmemory-policy` e LRU/LFU realmente se comportam.
- [Facebook: Scaling Memcache (cache stampede, thundering herd)](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf) — o artigo clássico sobre caching em escala.
