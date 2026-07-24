# Backend para Plataforma de Streaming (como a Netflix)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Projete o backend de um serviço de streaming de vídeo: ele ingere um upload bruto, o transcodifica em múltiplas resoluções e bitrates, publica essas renditions em uma CDN e serve aos clientes um manifesto que eles podem transmitir de forma adaptativa. A parte difícil não é nenhuma funcionalidade isolada — é orquestrar um pipeline assíncrono de longa duração, manter a reprodução suave sobre redes não confiáveis e rastrear milhões de sessões concorrentes sem martelar seu banco de dados. Você construirá o control plane (metadados, catálogo, estado de sessão) e o contrato do data plane (manifestos e URLs de segmento), tratando armazenamento e CDN como infraestrutura plugável. O foco é arquitetura e trade-offs, não entregar seu próprio codec.

## Pré-requisitos

- Design sólido de APIs REST e experiência com jobs/filas assíncronas (um projeto de message-broker ou worker-queue é um bom aquecimento)
- Familiaridade com cache HTTP, CDNs e range requests
- Conforto com object storage (compatível com S3) e workers de background
- Entendimento conceitual de vídeo: containers, codecs, bitrate, keyframes
- Um framework de sua escolha (Node, Go, Java, Python) além de qualquer transcoder que envolva o FFmpeg

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar um pipeline de transcodificação como estágios idempotentes, retomáveis e observáveis
- Explicar streaming de bitrate adaptativo e gerar manifestos HLS/DASH
- Projetar um store de catálogo/metadados que escala leituras independentemente das escritas
- Rastrear sessões de reprodução (posição de retomada, heartbeat) de forma barata em escala
- Raciocinar sobre offload de CDN, chaves de cache e URLs assinadas/com expiração
- Escolher trade-offs de consistência e disponibilidade para cada subsistema

## Requisitos Funcionais

1. O sistema deve aceitar um upload de vídeo e retornar um asset ID enquanto a transcodificação prossegue de forma assíncrona.
2. A transcodificação deve produzir múltiplas renditions (ex.: 240p–1080p) e segmentá-las para HLS e/ou DASH.
3. Cada estágio do pipeline deve ser idempotente e retomável para que um job repetido ou que caiu nunca duplique ou corrompa a saída.
4. O sistema deve servir um manifesto que lista as renditions disponíveis; clientes escolhem um bitrate de forma adaptativa.
5. A entrega de segmentos e manifestos deve poder ficar atrás de uma CDN, usando URLs assinadas e com expiração para controle de acesso.
6. O sistema deve persistir sessões de reprodução por usuário (posição de retomada, histórico) via heartbeats leves.
7. A API de catálogo deve permanecer disponível para navegação mesmo se o pipeline de transcodificação estiver degradado.
8. **Não funcional:** mire ≥99,9% de disponibilidade para reprodução e leituras de catálogo; sustente milhares de heartbeats concorrentes; mantenha baixa a latência de manifesto via cache; defina o comportamento sob falha parcial de CDN e backlog de workers.

## Marcos Sugeridos

1. **Marco 1 — Upload e metadados:** Aceite um upload para object storage, crie o registro do asset, enfileire um job de transcodificação.
2. **Marco 2 — Pipeline de transcodificação:** Rode workers em estágios (probe → transcodificar renditions → segmentar → gerar manifesto), cada um idempotente e com status rastreado.
3. **Marco 3 — Reprodução:** Sirva manifestos e URLs de segmento assinadas pela CDN; implemente heartbeats de sessão e retomada.
4. **Marco 4 — Escala e resiliência:** Adicione cache, retries com backoff, tratamento de dead-letter e métricas/dashboards para a saúde do pipeline.

## Esboço de Dados e Interface

```text
Componentes
  [Cliente] --upload--> [API de Ingestão] --> [Object Storage (bruto)]
                                        \--> [Fila de Jobs] --> [Workers de Transcodificação]
                                                                     |
                                        [Object Storage (renditions + segmentos)]
                                                                     |
  [Cliente] <--manifesto/segmentos-- [CDN] <---- origem ---- [API de Entrega]
  [Cliente] --heartbeat--> [API de Sessão] --> [KV / cache] --async--> [DB de Histórico]

Asset
  id, title, status(uploaded|transcoding|ready|failed),
  durationSec, renditions[{height, bitrate, codec, manifestUrl}]

GET  /catalog/{id}          -> metadados do asset
GET  /play/{id}/manifest    -> HLS(.m3u8) ou DASH(.mpd), URLs de segmento assinadas
POST /sessions/{id}/heartbeat  body:{ positionSec } -> 204
GET  /sessions/{id}         -> { resumePositionSec, updatedAt }
```

## Desafios Extras

- Adicione trilhas de legenda (WebVTT) referenciadas no manifesto.
- Implemente um feed de recomendação simples a partir do histórico de exibição.
- Adicione encoding por título (escolha a escada de bitrate com base na complexidade do conteúdo).
- Suporte transmissão ao vivo com uma janela de segmentos deslizante.

## Definição de Pronto

- [ ] Um upload transiciona pelos estados do pipeline até `ready` com múltiplas renditions produzidas.
- [ ] Um job de transcodificação repetido ou duplicado não produz saída duplicada ou parcial.
- [ ] Um player consegue buscar um manifesto HLS ou DASH válido e transmitir segmentos via URLs assinadas da CDN.
- [ ] A posição de retomada sobrevive a uma desconexão e é restaurada na próxima reprodução.
- [ ] Leituras de catálogo têm sucesso enquanto o pipeline de transcodificação está intencionalmente pausado.
- [ ] Métricas do pipeline (profundidade de fila, latência de estágio, taxa de falha) são observáveis.

## Armadilhas Comuns

- Tornar os estágios de transcodificação não idempotentes, de modo que um retry reanexa segmentos ou cobra armazenamento em dobro.
- Bloquear a requisição de upload na transcodificação em vez de retornar imediatamente e trabalhar de forma assíncrona.
- Escrever todo heartbeat direto no banco primário e derretê-lo sob carga — bufferize em um cache.
- Servir segmentos da origem em vez da CDN, anulando todo o propósito do cache de borda.
- Esquecer a expiração de URL/segmento, vazando acesso permanente a conteúdo protegido.

## Recursos

- [Apple: HTTP Live Streaming (HLS)](https://developer.apple.com/documentation/http-live-streaming) — a referência canônica de HLS.
- [Visão geral de MPEG-DASH (Bitmovin)](https://bitmovin.com/mpeg-dash-explained/) — manifestos DASH e streaming adaptativo.
- [Netflix TechBlog: Per-Title Encode Optimization](https://netflixtechblog.com/per-title-encode-optimization-7e99442b62a2) — trade-offs reais de escada de bitrate.
- [Documentação do FFmpeg](https://ffmpeg.org/documentation.html) — transcodificação e segmentação.
- [MDN: HTTP range requests](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Range_requests) — entrega de conteúdo parcial para mídia.
