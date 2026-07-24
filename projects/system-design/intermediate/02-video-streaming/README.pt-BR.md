# Projete um Sistema de Streaming de Vídeo

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Projete o backend de uma plataforma de vídeo sob demanda como Netflix ou YouTube: criadores enviam um arquivo, o sistema o transcodifica em várias qualidades, e milhões de espectadores o assistem sem travar em redes instáveis. Os problemas interessantes são o pipeline de transcodificação offline, a entrega com bitrate adaptativo sobre HTTP e servir petabytes de bytes de forma barata por uma CDN. Este é um exercício de projeto: você produz um documento de design, números de capacidade e diagramas — não código funcional.

## Pré-requisitos

- Entendimento de HTTP, armazenamento de objetos e como CDNs fazem cache
- Noção básica de codecs de vídeo, containers e bitrate
- Familiaridade com filas de mensagens e processamento de jobs em background
- Conforto para estimar armazenamento e banda em escala

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar um pipeline de transcodificação assíncrono guiado por uma fila de jobs
- Explicar o streaming com bitrate adaptativo (HLS/DASH) e o layout de manifesto/segmentos
- Estimar armazenamento por título entre renderizações e banda de saída no pico
- Projetar uma hierarquia de cache CDN + origem e raciocinar sobre taxas de acerto
- Justificar trade-offs entre pré-transcodificar todas as renderizações vs. sob demanda

## Requisitos e Restrições

- Assuma 100k novos vídeos/dia (média de 10 min), 50M de espectadores diários, 5M simultâneos no pico.
- A reprodução deve começar em menos de ~2 s e se adaptar suavemente a quedas de banda.
- Cada título é armazenado em ~5 renderizações (240p–1080p) mais miniaturas e legendas.
- Estime o armazenamento bruto + transcodificado por título e a saída total de pico em Tbps.
- Uploads e transcodificação são assíncronos; a visualização tem muitas leituras e é sensível à latência.

## Abordagem Sugerida

1. Separe o caminho de escrita (upload → transcodificação) do caminho de leitura (manifesto → segmentos).
2. Dimensione o armazenamento: tamanho bruto × renderizações × novos vídeos/dia; projete um ano.
3. Projete o pipeline de transcodificação: fatie a fonte, distribua os jobs, remonte.
4. Projete a entrega HLS/DASH: o manifesto aponta para segmentos cacheados na borda.
5. Planeje as camadas de CDN (borda → regional → origem) e como a popularidade guia o cache.

## Esboço de Arquitetura

```text
Upload -> API -> Store de Objetos Bruto (tipo S3)
                    |-> jobs de transcode -> Fila -> Frota de workers -> Store de Renderizações + Manifestos
                                                          |-> BD de metadados (particiona por videoId)

Espectador -> borda CDN -> cache regional -> origem (Store de Renderizações)
   GET /videos/{id}/master.m3u8   -> 200 manifesto (lista de renderizações)
   GET /videos/{id}/1080p/seg_{n}.ts -> 200 segmento (cacheado na borda)

POST /videos                 { title, ownerId } -> 201 { videoId, uploadUrl }
GET  /videos/{id}/play                          -> 200 { manifestUrl, subtitles[] }

Video { videoId, ownerId, status, durationSec, renditions[], ts } // particiona por videoId
Layout de segmentos: master.m3u8 -> {rendition}.m3u8 -> seg_0.ts ... seg_n.ts
```

## Tópicos de Aprofundamento

- **Pipeline de transcodificação:** paralelismo por chunk, retentativas, prioridade para uploads populares.
- **Bitrate adaptativo:** troca de renderização guiada pelo cliente; trade-offs de duração de segmento.
- **Trade-off 1 — pré-transcodificar tudo vs. sob demanda:** pré-transcodificar desperdiça armazenamento em títulos não assistidos, mas garante reprodução instantânea; sob demanda economiza armazenamento mas adiciona latência na primeira visualização. Justifique um híbrido (pré-transcodificar as renderizações principais, gerar o resto sob demanda).
- **Trade-off 2 — tamanho do segmento:** segmentos curtos se adaptam mais rápido a mudanças de banda, mas incham manifestos e a contagem de requisições.

## Entregáveis

- [ ] Um documento de design (~3–5 páginas) cobrindo os caminhos de escrita e leitura.
- [ ] Estimativas de capacidade: armazenamento por título, armazenamento total/ano, banda de saída de pico.
- [ ] Uma estratégia de cache CDN com taxa de acerto esperada e política de evicção.
- [ ] Um plano de particionamento de metadados.
- [ ] Pelo menos dois trade-offs, cada um com a opção escolhida e o porquê.

## Armadilhas Comuns

- Transcodificar o arquivo inteiro em um único job, fazendo um filme de 4 horas bloquear um worker por horas.
- Ignorar o custo de saída — banda, não armazenamento, geralmente domina a conta.
- Cachear manifestos tão agressivamente quanto segmentos, fazendo mudanças de renderização nunca se propagarem.
- Esquecer miniaturas, legendas e faixas de áudio na estimativa de armazenamento.

## Recursos

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — fundamentos de CDN e capacidade.
- [Visão geral de HLS (Apple, RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — o protocolo HTTP Live Streaming.
- [Netflix Tech Blog: encoding](https://netflixtechblog.com/) — transcodificação e entrega em escala no mundo real.
- [web.dev: Fast playback with audio and video preload](https://web.dev/articles/fast-playback-with-preload) — técnicas de latência de início.
