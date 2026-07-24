# Projete uma Plataforma de Streaming de Vídeo como a Netflix

> 🌐 [English](./README.md) · **Português**

**Domínio:** System Design · **Nível:** Avançado · **Tempo estimado:** 3–7 dias

## Visão Geral

Projete um serviço global de streaming de vídeo sob demanda no estilo da Netflix: os usuários navegam por um catálogo personalizado, apertam play e recebem um stream fluido e adaptativo que sobrevive a instabilidades de rede. A engenharia interessante não está no player — está em tudo por trás dele: ingerir um arquivo mestre, transcodificá-lo em dezenas de versões (bitrate/resolução), empurrar esses bytes para caches de borda próximos aos espectadores e servir uma home page cujas fileiras são ranqueadas por usuário em dezenas de milissegundos. Este é um exercício de projeto: sua entrega é um documento de design com diagramas e análise de trade-offs, não código em execução.

## Pré-requisitos

- Domínio sólido de HTTP, TCP e TLS, além de como CDNs e cache de borda funcionam
- Familiaridade com o ferramental de escala para leitura pesada (réplicas, caches, sharding)
- Conceitos básicos de vídeo: codecs, containers, bitrate adaptativo (HLS/DASH)
- Ter visto um design de nível mais baixo antes (veja os projetos intermediários em `../`) ajuda

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Estimar armazenamento, banda de egresso e compute de transcodificação para um catálogo em escala planetária
- Projetar um pipeline de transcodificação offline que produz uma escada ABR por título
- Raciocinar sobre posicionamento de CDN, taxas de acerto de cache e blindagem de origem
- Separar o caminho de personalização/ranqueamento do plano de dados de streaming
- Escolher modelos de consistência apropriados para metadados de catálogo versus histórico de visualização

## Requisitos e Restrições

1. Servir uma home page personalizada (fileiras de títulos) com latência p99 abaixo de ~200 ms.
2. Suportar streaming adaptativo que ajusta a versão à banda disponível sem rebuffering.
3. Mirar 99,99% de disponibilidade para reprodução; personalização degradada é aceitável, um player morto não é.
4. Assumir ~250M de assinantes, ~10% de concorrência de pico e um catálogo de múltiplos terabytes.
5. Aplicar DRM e restrições geográficas/de licenciamento por título e região.
6. Otimizar o custo de egresso — banda domina a conta nesta escala.
7. Capturar telemetria de visualização (pontos de retomada, QoE) sem bloquear a reprodução.

## Abordagem Sugerida

Comece pela matemática de capacidade: derive streams concorrentes de pico, bitrate médio e, portanto, egresso agregado (Tbps). Dimensione o armazenamento do catálogo depois de multiplicar cada título por sua escada de versões. Depois divida o sistema em três planos: um **plano de controle** (metadados de catálogo, direitos), um **plano de dados** (entrega de segmentos via CDN) e um **plano de inteligência** (recomendações, ranqueamento). Projete o pipeline de ingestão/transcodificação como um grafo de jobs assíncrono e idempotente. Trate a CDN como sua principal alavanca de escala e a origem como um fallback blindado. Pré-compute as fileiras personalizadas offline e sirva-as de um armazenamento rápido, atualizando em cadência em vez de por requisição.

## Esboço de Arquitetura

```text
Player do cliente ──> API Gateway ──> Svc Home/Ranking ──> Fileiras pré-computadas (cache KV)
                       │
                       ├──> Svc Catálogo ──> BD de metadados (replicado)
                       └──> Svc Direitos/DRM ──> servidor de licenças

Requisição de play: player ──> Svc Steering ──> PoP de CDN mais próximo ──> segmento (.ts/.m4s)
                                              └─ miss ─> Blindagem de origem ─> Object store (tipo S3)

Ingestão: mestre ──> Pipeline de transcodificação (grafo de jobs) ──> Escada ABR ──> Object store ──> CDN

APIs principais:
GET  /home?profile=P           -> { rows: [ {title, ranked_items[]} ] }
GET  /titles/{id}/manifest     -> manifesto HLS/DASH (por região, por dispositivo)
POST /playback/heartbeat       -> { titleId, positionMs, bitrate, droppedFrames }

Modelo de dados (esboço):
Title{ id, metadata, availabilityByRegion[], renditions[] }
Rendition{ resolution, bitrate, codec, segmentUri }
ViewHistory{ profileId, titleId, positionMs, updatedAt }  # última escrita vence
```

## Tópicos de Aprofundamento

- **Design da escada ABR:** quantas versões, quais resoluções/bitrates, codificação por título.
- **Economia da CDN:** hierarquia de cache, metas de taxa de acerto, blindagem de origem, caches embutidos em ISPs.
- **Personalização em escala:** ranqueamento batch offline vs re-ranqueamento online; usuários em cold-start.
- **Divisão de consistência:** quase forte para direitos, eventual para histórico e ranqueamentos.
- **DRM e licenciamento:** entrega de chaves, aplicação de bloqueios regionais, URLs de segmento assinadas por token.

## Entregáveis

- [ ] Um documento de design (~4–8 páginas) com o diagrama de arquitetura acima, refinado.
- [ ] Estimativas aproximadas de capacidade para armazenamento, egresso de pico e compute de transcodificação.
- [ ] Escolhas explícitas de consistência por domínio de dados, com justificativa.
- [ ] Uma análise de falha/DR: queda de PoP, perda de região, falha de origem, backlog de transcodificação.
- [ ] Gargalos identificados e mitigações de hotspot (ex.: efeito manada de um lançamento viral).

## Armadilhas Comuns

- Rotear cada byte de reprodução pela sua origem em vez da CDN — a conta e a origem explodem.
- Recomputar fileiras personalizadas de forma síncrona por requisição, estourando o orçamento de latência.
- Tratar escritas de histórico como fortemente consistentes; elas devem ser assíncronas e idempotentes.
- Ignorar o backlog de transcodificação: uma grande importação de catálogo pode travar a codificação por dias.
- Esquecer o licenciamento regional, fazendo um título reproduzir onde não é licenciado.

## Recursos

- [Netflix Tech Blog](https://netflixtechblog.com/) — relatos em primeira mão da arquitetura real.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — fundamentos de CDN, cache e escala.
- [HTTP Live Streaming (RFC 8216)](https://datatracker.ietf.org/doc/html/rfc8216) — a especificação de streaming adaptativo HLS.
- [Open Connect (CDN da Netflix)](https://openconnect.netflix.com/en/) — como a Netflix embute caches dentro de ISPs.
- [Visão geral do MPEG-DASH (MDN)](https://developer.mozilla.org/pt-BR/docs/Web/Media/DASH_Adaptive_Streaming_for_HTML_5_Video) — streaming adaptativo na web.
