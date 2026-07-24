# Projete um Feed de Rede Social

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend que produz o feed inicial de uma rede como Twitter/X ou Instagram: usuários seguem outros, publicam posts e abrem o app para um fluxo ranqueado de atividade recente. A tensão definidora é o fan-out — quando alguém com milhões de seguidores posta, você empurra esse post para milhões de feeds imediatamente ou monta cada feed na leitura? A resposta molda todo o sistema. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento do grafo de seguidores como modelo de dados
- Familiaridade com estratégias de cache e pré-computação
- Noção de filas de mensagens para fan-out assíncrono
- Conforto para estimar razões leitura/escrita e armazenamento de timelines

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Comparar fan-out-na-escrita vs. fan-out-na-leitura e escolher por classe de usuário
- Estimar QPS de leitura do feed, QPS de escrita de posts e armazenamento do cache de timelines
- Projetar um feed híbrido que trate usuários normais e celebridades
- Particionar o grafo de seguidores e os stores de timeline de forma sensata
- Justificar trade-offs entre amplificação de escrita e latência de leitura

## Requisitos e Restrições

- Assuma 200M de usuários diários, ~5k posts/s, cada usuário abrindo o feed ~10×/dia (~20k leituras de feed/s).
- O carregamento do feed deve retornar em menos de ~200 ms; frescor quase em tempo real (segundos) basta.
- Algumas contas têm dezenas de milhões de seguidores (o "problema da celebridade").
- Os feeds mostram posts recentes de quem se segue, ranqueados (recência + sinais de engajamento).
- Estime a amplificação de escrita do fan-out e o armazenamento das timelines cacheadas.

## Abordagem Sugerida

1. Calcule o custo do fan-out: média de seguidores × taxa de posts = escritas de timeline/s.
2. Escolha fan-out-na-escrita para a maioria; recorra ao merge na leitura para celebridades.
3. Projete o cache de timeline (lista de IDs de post por usuário) e sua política de poda.
4. Adicione uma etapa de ranqueamento que reordena o conjunto candidato antes de retornar.
5. Particione o grafo e as timelines por userId; planeje para nós quentes de celebridades.

## Esboço de Arquitetura

```text
Post -> [svc Post] -> Store de Posts (shard por postId)
                         |-> worker de fan-out -> Fila -> escreve postId nas timelines dos seguidores (cache)
                                                          (pula se o autor for celebridade)

Abrir app -> [svc Feed] -> lê o cache da própria timeline
                            + faz merge dos posts recentes das celebridades seguidas (na leitura)
                            -> Ranqueamento -> retorna página

POST /posts            { userId, text, media[] } -> 201 { postId }
GET  /feed?userId&cursor                          -> 200 { posts[], nextCursor }
POST /follow           { followerId, followeeId } -> 204

Follow   { followerId, followeeId, ts }        // particiona por followerId
Timeline { userId -> [postId, ...] }           // cache por usuário, podado a N
Post     { postId, authorId, text, ts, stats } // shard por postId
```

## Tópicos de Aprofundamento

- **Estratégia de fan-out:** push (escrita) vs. pull (leitura) vs. híbrido por limiar de contagem de seguidores.
- **Ranqueamento:** baseline por recência mais features de engajamento; mantenha barato na leitura.
- **Trade-off 1 — amplificação de escrita vs. leitura:** fan-out-na-escrita torna as leituras O(1) mas um post de celebridade dispara milhões de escritas; fan-out-na-leitura é barato de escrever mas lento de montar. Justifique um limiar híbrido.
- **Trade-off 2 — armazenamento de timeline:** cachear posts completos é rápido mas enorme; cacheie só IDs de post e hidrate a partir do store de posts na leitura.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo a arquitetura acima.
- [ ] Estimativas de capacidade: QPS de leitura do feed, QPS de escrita de posts, escritas de fan-out/s, armazenamento do cache de timelines.
- [ ] Um plano de particionamento para o grafo de seguidores e timelines, tratando nós quentes de celebridades.
- [ ] Uma estratégia de cache para timelines com política de poda/evicção.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Fan-out-na-escrita puro sem exceção para celebridades, fazendo um post viral travar o pipeline.
- Cachear corpos completos de posts por seguidor, multiplicando o armazenamento pela contagem de seguidores.
- Nunca podar timelines, fazendo usuários inativos acumularem histórico ilimitado.
- Ranquear sincronamente sobre milhares de candidatos, estourando o orçamento de latência.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — design de fan-out e timeline.
- [Twitter Engineering: a infraestrutura por trás das Timelines](https://blog.twitter.com/engineering/en_us/topics/infrastructure) — arquitetura de feed no mundo real.
- [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) — encaixe natural para timelines ranqueadas.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — o estudo de caso de fan-out do Twitter.
