# Projete uma Plataforma de Compartilhamento de Vídeo como o YouTube

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete uma plataforma de vídeo gerado por usuários no estilo do YouTube: qualquer um pode fazer upload, o sistema transcodifica e publica globalmente, e os espectadores assistem, buscam e recebem recomendações do que ver a seguir. Diferente de um catálogo curado, o lado da ingestão é uma mangueira — centenas de horas enviadas por minuto — e o lado da leitura é planetário. O design deve equilibrar um pipeline de upload ilimitado, um índice de busca sobre bilhões de itens, um motor de recomendação e entrega via CDN. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Entendimento de CDNs, armazenamento de objetos e streaming adaptativo (HLS/DASH)
- Familiaridade com pipelines assíncronos e filas de mensagens
- Exposição a conceitos de indexação de busca (índice invertido, ranqueamento)
- Entendimento básico de sistemas de recomendação

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de upload retomável e em chunks que alimenta um fan-out de transcodificação assíncrono
- Estimar o crescimento do armazenamento de ingestão e o egresso de leitura em escala global
- Projetar um índice de busca que permanece atualizado enquanto novos vídeos chegam continuamente
- Separar a agregação de contagem de visualizações do caminho quente de leitura
- Raciocinar sobre a latência de serviço de recomendação e o trade-off frescor/custo

## Requisitos e Restrições

1. Aceitar uploads de tamanho arbitrário com transferência retomável e em chunks.
2. Transcodificar cada upload em uma escada ABR de forma assíncrona; publicar quando pronto.
3. Servir reprodução via CDN com latência de início p99 abaixo de ~1 s.
4. Fornecer busca sobre bilhões de vídeos com latência de consulta sub-segundo.
5. Agregar contagens de visualização e engajamento sem bloquear a reprodução (consistência eventual OK).
6. Assumir ~500 horas enviadas/minuto e bilhões de visualizações diárias.
7. Suportar moderação de conteúdo e correspondência de direitos autorais no caminho de ingestão.

## Abordagem Sugerida

Separe o plano de ingestão pesado de escrita do plano de serviço pesado de leitura. Uploads chegam como chunks no armazenamento de objetos; um evento de conclusão dispara jobs de transcodificação (idempotentes, retentáveis) que produzem a escada ABR e miniaturas. Metadados publicam para um índice de busca de forma assíncrona. No lado da leitura, trate a CDN como o cavalo de batalha da entrega e pré-compute candidatos de recomendação offline, re-ranqueando um pequeno conjunto online. Contagens de visualização são o hotspot clássico: acumule incrementos em um stream e agregue-os, aceitando consistência eventual, para que um vídeo viral não martele uma única linha. Rode moderação/direitos autorais como verificações assíncronas que podem retirar um vídeo após a publicação.

## Esboço de Arquitetura

```text
Uploader ──chunks──> Svc Upload ──> Object store (bruto) ──evento──> Fan-out transcodificação (fila)
                                                                        │
                                                          Escada ABR + miniaturas -> Object store -> CDN
                                                                        │
                                                          Metadados -> Índice de busca (async) + BD Catálogo

Assistir: espectador ──> API ──> Catálogo ──> manifesto ──> segmentos CDN
Contagem de views: player ──beacon──> stream ──agrega──> store de contagens (eventual)
Recs: geração de candidatos offline -> feature store -> re-rank online -> feed

APIs principais:
POST /uploads (retomável)          -> { uploadId, uploadUrls[] }
GET  /videos/{id}                  -> { metadata, manifestUrl, status }
GET  /search?q=...                 -> { results[], nextPage }
POST /videos/{id}/view             -> 202 (contagem async)

Modelo de dados (esboço):
Video{ id, ownerId, status: PROCESSING|LIVE|BLOCKED, renditions[], stats }
Stats{ views, likes, watchTimeMs }   # agregados eventualmente consistentes
```

## Tópicos de Aprofundamento

- **Uploads retomáveis:** chunking, retentativa, verificação de checksum, multipart para armazenamento de objetos.
- **Fan-out de transcodificação:** filas de prioridade, idempotência, tratar a cauda longa de formatos.
- **Frescor da busca:** indexação quase em tempo real vs batch; sinais de ranqueamento.
- **Hotspots de contagem de views:** agregação por stream, contadores particionados, contagem aproximada.
- **Moderação e direitos autorais:** fingerprinting de conteúdo, takedown assíncrono, retração pós-publicação.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com os planos de ingestão e serviço separados, refinado.
- [ ] Estimativas de capacidade para crescimento do armazenamento de ingestão, compute de transcodificação e egresso de leitura.
- [ ] A estratégia de agregação de contagem de views com seu trade-off de consistência declarado.
- [ ] Uma análise de falha/DR: backlog de transcodificação, atraso do índice de busca, perda de PoP de CDN.
- [ ] Um plano de mitigação de hotspot para um vídeo viralizar em minutos.

## Armadilhas Comuns

- Bloquear a conclusão do upload na transcodificação em vez de publicar "processando" e terminar async.
- Incrementar uma única linha de contagem de views por visualização — um vídeo viral cria um hotspot de escrita.
- Tornar a busca sincronamente consistente com os uploads, acoplando duas cargas muito diferentes.
- Servir reprodução da origem em vez da CDN, explodindo o custo de egresso.
- Pular idempotência em jobs de transcodificação, fazendo retentativas produzirem versões duplicadas.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — pipelines, CDNs e busca em escala.
- [Google Research: o sistema de recomendação do YouTube](https://research.google/pubs/pub45530/) — deep learning para recomendações.
- [Netflix Tech Blog](https://netflixtechblog.com/) — práticas de transcodificação e streaming adaptativo que se transferem diretamente.
- [HTTP Live Streaming (RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — a especificação HLS para entrega.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — processamento de streams e agregação.
