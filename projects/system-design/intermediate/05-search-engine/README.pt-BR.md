# Projete um Motor de Busca

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de um motor de busca de texto completo que rastreia ou ingere documentos, constrói um índice invertido e responde a consultas ranqueadas por palavras-chave em milissegundos. Pense em uma busca de site ou um índice web de menor escala. O coração do sistema é o índice invertido — como você o constrói, faz sharding, mantém atualizado e consulta em muitas máquinas. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento do índice invertido e da tokenização
- Familiaridade com pontuação de relevância TF-IDF ou BM25 em nível conceitual
- Noção de sharding, replicação e consultas scatter-gather
- Conforto para estimar o tamanho do índice em relação ao corpus

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de indexação: buscar → parsear → tokenizar → postar no índice
- Estimar tamanho do corpus, tamanho do índice invertido e QPS de consultas
- Fazer sharding do índice (por documento vs. por termo) e rodar consultas scatter-gather
- Projetar uma camada de cache para consultas populares e um plano de frescor do índice
- Justificar trade-offs entre índices particionados por documento e por termo

## Requisitos e Restrições

- Assuma 500M de documentos (média de 5 KB de texto), ~10k consultas/s no pico, índice atualizado continuamente.
- A latência de consulta deve ficar abaixo de ~200 ms no p99, incluindo o ranqueamento.
- Documentos novos/atualizados devem ficar buscáveis em minutos.
- Os resultados são ranqueados por relevância; suporte filtros básicos (data, tipo).
- Estime o armazenamento bruto do corpus, o tamanho do índice invertido e o throughput de consultas.

## Abordagem Sugerida

1. Projete o pipeline de ingestão e como os documentos fluem para segmentos.
2. Estime o tamanho do índice (listas de postings) em relação ao corpus de 500M de documentos.
3. Escolha um esquema de particionamento: shards particionados por documento com scatter-gather.
4. Adicione um cache de consultas e um cache de resultados para consultas mais frequentes.
5. Projete indexação quase em tempo real: pequenos segmentos mesclados em background.

## Esboço de Arquitetura

```text
Crawler/Ingestão -> Parser/Tokenizador -> [Indexador] -> Escritor de segmentos -> Shard 1..N (índice invertido)
                                                             |-> merge/compactação em background

Consulta -> [svc Consulta] -> cache de consulta? -> espalha para todos os shards -> junta top-k -> ranqueia (BM25) -> merge
                                                                                       |-> cache de resultados

GET /search?q=sistemas+distribuidos&page=1 -> 200 { results[], total, tookMs }

Document { docId, url, title, body, ts }        // particiona por hash de docId (por documento)
Postings { term -> [(docId, tf, positions), ...] }  // índice invertido por shard
```

## Tópicos de Aprofundamento

- **Construção do índice:** tokenização, stemming, stop words; escritas baseadas em segmentos e merges.
- **Ranqueamento:** pontuação BM25, combinando frequência de termo com frequência de documento entre shards.
- **Trade-off 1 — particionar por documento vs. por termo:** por documento espalha cada consulta a todos os shards mas as escritas são locais e simples; por termo roteia uma consulta a poucos shards mas faz indexar um documento tocar muitos shards. Justifique o particionamento por documento para carga balanceada.
- **Trade-off 2 — frescor vs. throughput:** segmentos pequenos e frequentes mantêm resultados frescos mas prejudicam a velocidade da consulta até o merge; merges em lote são eficientes mas atrasam a visibilidade.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) expandindo a arquitetura acima.
- [ ] Estimativas de capacidade: armazenamento do corpus, tamanho do índice invertido, QPS de consultas, número de shards.
- [ ] Um plano de particionamento (por documento vs. por termo) com justificativa.
- [ ] Uma estratégia de cache para caches de consulta e de resultados, com invalidação em atualizações.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Reconstruir o índice inteiro a cada atualização em vez de escrever segmentos incrementais.
- Ignorar a cauda do scatter-gather: o shard mais lento define a latência da consulta.
- Subestimar o tamanho do índice — posições e metadados podem rivalizar com o texto bruto.
- Cachear resultados sem invalidá-los quando os documentos subjacentes mudam.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — sharding e scatter-gather.
- [Elasticsearch: o que é um índice invertido](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up) — internos do índice explicados.
- [BM25 (Wikipedia)](https://en.wikipedia.org/wiki/Okapi_BM25) — a função padrão de pontuação de relevância.
- [Introduction to Information Retrieval (Manning et al.)](https://nlp.stanford.edu/IR-book/) — o texto de referência para indexação e ranqueamento.
