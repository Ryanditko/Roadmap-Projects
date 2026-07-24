# Projete um Sistema de Cache

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Projete uma camada de cache que fica entre uma aplicação e um store de apoio lento, servindo dados quentes da memória para cortar latência e carga. O cerne do problema não é armazenar dados — é decidir o que remover quando a memória enche e como impedir que valores em cache fiquem obsoletos. Você vai raciocinar sobre políticas de eviction, padrões de leitura/escrita e os perigos clássicos: leituras obsoletas, cache stampedes e o thundering herd sobre um cache frio. Entregue um documento de design cobrindo o posicionamento do cache, a política de eviction e a estratégia de invalidação.

## Pré-requisitos

- Entendimento de que memória é rápida e pequena, disco/BD é lento e grande
- Consciência do que são um cache hit e um cache miss
- Familiaridade com a ideia de TTL (time to live)
- Conforto para raciocinar sobre consistência entre um cache e sua fonte da verdade

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escolher uma política de eviction (LRU, LFU, TTL) e justificá-la
- Comparar padrões de cache de leitura/escrita (cache-aside, write-through)
- Raciocinar sobre obsolescência e invalidação
- Estimar a taxa de acerto necessária para atingir uma meta de latência
- Enunciar um trade-off entre frescor e desempenho

## Requisitos e Restrições

1. Servir leituras do cache em um hit; cair para o store em um miss.
2. Remover entradas quando o cache atinge seu limite de memória.
3. Manter valores razoavelmente frescos — obsolescência limitada via TTL ou invalidação.
4. Tratar um cache miss sem derrubar o store de apoio sob um stampede.
5. Expor métricas de hit/miss para que a eficácia seja observável.
6. Estime a escala: 100K leituras/s, working set de 10 GB, taxa de acerto alvo ≥ 90%.

## Abordagem Sugerida

1. Decida o padrão de cache: cache-aside (a app gerencia o cache) é o padrão comum.
2. Faça a conta: a 100K leituras/s e 90% de hit rate, só ~10K leituras/s chegam ao store — essa é a carga que você precisa sobreviver.
3. Escolha uma política de eviction e raciocine sobre qual se encaixa no padrão de acesso.
4. Projete a invalidação: expiração por TTL, invalidação explícita na escrita ou write-through.
5. Trate o stampede: quantas requisições atingem o store quando uma chave quente expira, e como amortecer isso.

## Esboço de Arquitetura

```text
             leitura
Cliente ──> [ App ] ──> [ Cache ] --hit--> valor
                           │ miss
                           ▼
                      [ Store ] -> popula cache -> valor

Padrões
  Cache-aside     app lê cache, no miss carrega do store e escreve no cache
  Write-through   escritas vão para cache e store juntos (mais fresco, escritas lentas)
  Read-through    o próprio cache carrega do store no miss

Eviction
  LRU  remove o menos recentemente usado   LFU  remove o menos frequentemente usado
  TTL  expira após tempo fixo

Modelo de dados
  entrada de cache: key | value | expires_at | last_access
```

## Tópicos de Aprofundamento

- **Políticas de eviction:** quando LRU vence vs. LFU, e por que só TTL pode servir dado obsoleto.
- **Cache stampede:** locking, coalescência de requisições ou expiração antecipada probabilística para impedir muitos misses atingirem o store de uma vez.
- **Consistência:** a corrida de escrita obsoleta do cache-aside e como write-through ou invalidação a reduz.

## Entregáveis

- Um diagrama de arquitetura mostrando o posicionamento do cache e o caminho de miss até o store.
- O padrão de leitura/escrita e a política de eviction escolhidos, com uma breve justificativa.
- Um modelo de dados para uma entrada de cache.
- Um trade-off descrito: ex., write-through (sempre fresco, escritas mais lentas) vs. cache-aside com TTL (escritas rápidas, janela de obsolescência limitada).

## Armadilhas Comuns

- Projetar o armazenamento mas nunca definir a política de eviction, deixando o comportamento na capacidade máxima indefinido.
- Ignorar o stampede: uma chave popular expirando deixa milhares de misses atingirem o store de uma vez.
- Cachear sem um plano de invalidação, deixando escritas deixarem valores obsoletos legíveis indefinidamente.
- Não reportar métricas de hit/miss, sem saber se o cache ajuda em algo.

## Recursos

- [System Design Primer: Cache](https://github.com/donnemartin/system-design-primer#cache) — padrões e posicionamento de cache.
- [Redis: Políticas de eviction de chaves](https://redis.io/docs/latest/develop/reference/eviction/) — como um cache real remove entradas.
- [AWS: Estratégias de caching](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html) — cache-aside vs. write-through.
- [Wikipedia: Cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) — o problema do thundering-herd e mitigações.
